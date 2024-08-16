```ts
import axios, { AxiosResponse, AxiosRequestConfig } from "axios";
import { stringify } from "qs";
import moment from "moment";
import { saveAs } from "file-saver";

import { decodeJwt, isObject } from "utils/helperFunctions";

// Create an Axios instance with a default base URL
const api = axios.create({
  baseURL: `${process.env.REACT_APP_BACKEND_URL}`,
  headers: {
    "Content-Type": "application/json",
  },
  timeout: 1000 * 120, // 120 seconds
});

// Add the 401 response interceptor
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error?.response?.status === 401) {
      // Handle logout
    } else {
      return Promise.reject(error);
    }
  }
);

export default class ApiClient {
  /**
   *
   * @param url
   * @param params
   */

  // Converts a query object or string to a URL query string
  private static convertQuery(url: string, query: any): string {
    if (!query) return url;
    if (typeof query === "string" && query.trim()) return `${url}?${query}`;
    if (isObject(query) && Object.keys(query).length > 0)
      return `${url}?${stringify(query)}`;
    return url;
  }

  // Get accessToken
  private static getToken(): string {
    const timeNow = moment();
    const accessToken = localStorage.getItem("token");
    if (!accessToken) {
      return "";
    }

    // Check token is expired
    const expiredToken = decodeJwt(accessToken)?.exp;
    const isTokenExpired = moment.unix(expiredToken ?? 0).isBefore(timeNow);
    if (!isTokenExpired) {
      return `Bearer ${accessToken}`;
    } else {
      return "";
    }
  }

  // Builds headers for different content types
  private static getHeaders(contentType = "application/x-www-form-urlencoded") {
    return {
      "Content-Type": contentType,
      authorization: this.getToken(),
    };
  }

  private static convertToPostData(obj: any, form: any, namespace: any) {
    const fd = form || new URLSearchParams();
    let formKey;

    for (const property in obj) {
      if (obj.hasOwnProperty(property)) {
        if (namespace) {
          if (!isNaN(Number(property))) {
            formKey = `${namespace}[${property}]`;
          } else {
            formKey = `${namespace}.${property}`;
          }
        } else {
          formKey = property;
        }

        if (obj[property] instanceof Date) {
          fd.append(formKey, obj[property].toISOString());
        } else if (
          Array.isArray(obj[property]) &&
          obj[property].every((item: any) => item instanceof File)
        ) {
          for (const i of Object.keys(obj[property])) {
            fd.append(formKey, obj[property][i]);
          }
        } else if (
          typeof obj[property] === "object" &&
          !(obj[property] instanceof File) &&
          !(obj[property] instanceof Blob)
        ) {
          this.convertToPostData(obj[property], fd, formKey);
        } else {
          fd.append(formKey, obj[property]);
        }
      }
    }

    return fd;
  }

  static async get<T = any>(
    url: string,
    params: object,
    query?: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);
    const response = await api.get<T>(requestUrl, {
      params,
      headers: this.getHeaders("application/json"),
      data: {},
    });
    return response;
  }

  static async getNoHeader<T = any>(
    url: string,
    params: object,
    query?: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);
    const response = await api.get<T>(requestUrl, {
      params,
      headers: {
        "Content-Type": "application/json",
      },
      data: {},
    });
    return response;
  }

  static async post<T = any>(
    url: string,
    query: any,
    params: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);

    const config: AxiosRequestConfig = {
      headers: this.getHeaders("application/json"),
    };

    const param = this.convertToPostData(params, undefined, undefined);
    const response = await api.post<T>(requestUrl, param, config);
    return response;
  }

  static async postNoHeader<T = any>(
    url: string,
    query: any,
    params: any,
    extraConfig?: Partial<AxiosRequestConfig>
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);

    const config: AxiosRequestConfig = {
      headers: {
        "Content-Type": "application/json",
      },
      ...extraConfig,
    };

    const response = await api.post<T>(requestUrl, params, config);
    return response;
  }

  static async put<T = any>(
    url: string,
    query: any,
    params: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);

    const config: AxiosRequestConfig = {
      headers: this.getHeaders("application/json"),
    };

    const response = await api.put<T>(requestUrl, params, config);
    return response;
  }

  static async delete<T = any>(
    url: string,
    query: any,
    params?: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);
    const config: AxiosRequestConfig = {
      headers: this.getHeaders("application/json"),
      ...(params ? { data: params } : {}),
    };
    const response = await api.delete<T>(requestUrl, config);
    return response;
  }

  static async postMultipartData(
    url: string,
    query: any,
    params: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);

    const config: AxiosRequestConfig = {
      headers: this.getHeaders("multipart/form-data"),
    };
    const form = new FormData();
    const param = this.convertToPostData(params, form, undefined);

    const response = await api.post(requestUrl, param, config);

    return response;
  }

  static async downloadExcelPost(
    url: string,
    query: any,
    params: object,
    fileName = "excel_table",
    isFinished = true
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);

    const config: AxiosRequestConfig = {
      headers: this.getHeaders("application/json"),
      responseType: "blob",
    };

    const response = await api.post(requestUrl, params, config);
    if (isFinished) {
      const blob = new Blob([response.data], {
        type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
      });
      saveAs(blob, fileName);
    }
    return response;
  }

  static async downloadExcelGet(
    url: string,
    params: object,
    query?: any,
    fileName?: string
  ) {
    const requestUrl = this.convertQuery(url, query);

    const response = await api.get(requestUrl, {
      params,
      headers: this.getHeaders("application/xlsx"),
      responseType: "blob",
      data: {},
    });

    const blob = new Blob([response.data], {
      type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    });
    saveAs(blob, fileName);
    return response;
  }

  static async downloadExcelGetXlsm(
    url: string,
    params: object,
    query?: any,
    fileName?: string
  ) {
    const requestUrl = this.convertQuery(url, query);

    const response = await api.get(requestUrl, {
      params,
      headers: this.getHeaders("application/vnd.ms-excel"),
      responseType: "blob",
      data: {},
    });

    const blob = new Blob([response.data], {
      type: "application/vnd.ms-excel",
    });
    saveAs(blob, fileName);
    return response;
  }

  static async uploadFileExcelPostMultipartData(
    url: string,
    query: any,
    params: any,
    option?: any
  ): Promise<AxiosResponse> {
    const requestUrl = this.convertQuery(url, query);
    const config: AxiosRequestConfig = {
      headers: this.getHeaders("multipart/form-data"),
      onUploadProgress: option?.onUploadProgress,
    };
    const response = await api.post(requestUrl, params, config);

    return response;
  }
}
```
