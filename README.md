
# 📊 Report Management System Documentation

## 📑 Mục lục
- [📂 Schemas](#schemas)
  - [📄 Report Info Schema](#report-info-schema)
  - [📄 Parameter Schema](#parameter-schema)
- [📝 API Documentation](#api-documentation)
  - [📊 Parameter API](#parameter-api)
    - [➕ POST /v1/pms/reports/:reportId/parameter](#-post-v1pmsreportsreportidparameter)
    - [🔍 GET /v1/pms/reports/:reportId/parameter/:id](#-get-v1pmsreportsreportidparameterid)
    - [✏️ PUT /v1/pms/reports/:reportId/parameter/:id](#️-put-v1pmsreportsreportidparameterid)
    - [🗑️ DELETE /v1/pms/reports/:reportId/parameter/:id](#️-delete-v1pmsreportsreportidparameterid)
    - [🔍 GET /v1/pms/reports/:reportId/parameter](#-get-v1pmsreportsreportidparameter)
  - [📊 Report API](#report-api)
    - [➕ POST /v1/pms/reports](#-post-v1pmsreports)
    - [🔍 GET /v1/pms/reports/:reportId](#-get-v1pmsreportsreportid)
    - [✏️ PUT /v1/pms/reports/:reportId](#️-put-v1pmsreportsreportid)
    - [🗑️ DELETE /v1/pms/reports/:reportId](#️-delete-v1pmsreportsreportid)
    - [🔍 GET /v1/pms/reports](#-get-v1pmsreports)

---

## Schemas

### Report Info Schema

- **Mô tả**: Lưu trữ thông tin báo cáo, bao gồm chủ sở hữu, tên báo cáo, và các phần tử trong báo cáo như `text`, `parameter` (biến dữ liệu để hiển thị giá trị khi xem lại), và `widget` (các đồ thị hoặc biểu đồ).

```javascript
const ReportInfoSchema = new Schema(
  {
    owner: {
      type: {
        userId: { type: String, required: true },
        fullName: { type: String, required: true },
        email: { type: String, required: true },
        role: {
          type: String,
          required: true,
          enum: ["admin", "member"],
          default: "member",
        },
        orgIdName: { type: String, required: true },
      },
      required: true,
    },
    reportName: {
      type: String,
      required: true,
      trim: true,
    },
    reportNormalizedName: {
      type: String,
      required: true,
      trim: true,
    },
    canvasInfo: {
      type: [
        {
          size: { type: Number, required: true },
          id: { type: String, required: true },
          x: { type: Number, required: true },
          y: { type: Number, required: true },
          width: { type: Number, required: true },
          height: { type: Number, required: true },
          fill: { type: String, default: "transparent" },
          color: { type: String, required: true },
          fontStyle: { type: String, required: true },
          align: { type: String, required: true },
          name: { type: String, required: true },
          uuid: { type: String, required: true },

          // other
          paramId: { type: String, required: true },
          elementType: { type: String, required: true, enum: ["parameter", "text", "widget"] },
        },
      ],
      default: [],
    },
  },
  { timestamps: true }
);
```

---

### Parameter Schema

- **Mô tả**: Lưu trữ biến dữ liệu của một báo cáo, giúp hiển thị danh sách dữ liệu có thể thêm vào báo cáo và cách chèn dữ liệu vào báo cáo.

```javascript
const ParameterSchema = new mongoose.Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    orgIdName: { type: String, required: true },
    kpiItem: {
      type: {
        displayName: String,
        unit: String,
        formula: String,
        isRaw: {
          type: Boolean,
        },
        isConstant: {
          type: Boolean,
          default: false,
        },
        params: [
          {
            name: String,
            variableName: String,
            deviceName: String,
            deviceType: {
              type: String,
              enum: ["E", "G", "W", "INVSOL", "SENSOL"],
            },
            aggregationType: {
              type: String,
              enum: ["sum", "avg", "max", "min", "last", "counter", null],
            },
          },
        ],
      },
      required: true,
    },
    startTS: { type: Number, required: true },
    endTS: { type: Number, required: true },
    name: { type: String },
    dataType: { type: String },
  },
  { timestamps: true }
);
```

---

## API Documentation

> Note: Tất cả các api cần header: { 'platform' : 'pms'} và cookies: { 'pms' : 'replaceCookie' }

### Parameter API

API cho `Parameter` giúp quản lý các biến dữ liệu của báo cáo. Hỗ trợ các thao tác tạo mới, cập nhật, lấy thông tin và xóa parameter.

#### ➕ POST /v1/pms/reports/:reportId/parameter

- **Description**: Tạo một parameter mới cho báo cáo.

- **Params**:
  `reportId`: id báo cáo

- **Body**:

```json
{
    "startTS": number,
    "endTS": number,
    "kpiItem": { 
        "displayName": string,
        "unit": string,
        "formula": string or null,
        "isRaw": boolean,
        "isConstant": boolean,
        "params": [
            {
                "name": string,
                "variableName": string,
                "deviceName": string,
                "deviceType": string,
                "aggregationType": string
            }
        ]
    }
}
```

- **Response**: (case success)

```json
{
    "message_en": "Parameter created successfully",
    "message_vn": "Tạo biến dữ liệu thành công",
    "item": {
        "reportId": "672c286b0ddb120c9e8c52be",
        "orgIdName": "bia-hanoi",
        "kpiItem": {
            "displayName": "vol_SN",
            "unit": "kW",
            "formula": null,
            "isRaw": true,
            "isConstant": false,
            "params": [
                {
                    "name": "vol_SN",
                    "variableName": "",
                    "deviceName": "H 30",
                    "deviceType": "E",
                    "aggregationType": "avg",
                    "_id": "672c28720ddb120c9e8c52c4"
                }
            ],
            "_id": "672c28720ddb120c9e8c52c3"
        },
        "startTS": 123123123,
        "endTS": 12312312313,
        "_id": "672c28720ddb120c9e8c52c2",
        "createdAt": "2024-11-07T02:39:46.071Z",
        "updatedAt": "2024-11-07T02:39:46.071Z",
        "__v": 0
    }
}
```

#### 🔍 GET /v1/pms/reports/:reportId/parameter/:id

- **Description**: Lấy thông tin parameter theo `id`.

- **Params**:
  `reportId`: id báo cáo
  `id`: id parameter

- **Response**:

```json
{
    "message_en": "Get detail parameter successfully",
    "message_vn": "Lấy chi tiết biến dữ liệu thành công",
    "items": {
        "_id": "672c3a4a027ef7f660e5ae5a",
        "reportId": "672c3a39027ef7f660e5ae56",
        "orgIdName": "bia-hanoi",
        "kpiItem": {
            "displayName": "vol_SN",
            "unit": "kW",
            "formula": null,
            "isRaw": true,
            "isConstant": false,
            "params": [
                {
                    "name": "vol_SN",
                    "variableName": "",
                    "deviceName": "H 30",
                    "deviceType": "E",
                    "aggregationType": "avg",
                    "_id": "672c3a4a027ef7f660e5ae5c"
                }
            ],
            "_id": "672c3a4a027ef7f660e5ae5b"
        },
        "startTS": 123123123,
        "endTS": 12312312313,
        "createdAt": "2024-11-07T03:55:54.208Z",
        "updatedAt": "2024-11-07T03:55:54.208Z",
        "__v": 0
    }
}
```

#### ✏️ PUT /v1/pms/reports/:reportId/parameter/:id 

- **Description**: Cập nhật thông tin parameter.

- **Params**:
  `reportId`: id báo cáo
  `id`: id parameter

- **Body**:

```json
{
    "startTS": number, // Có thể push tất cả hoặc đẩy thiếu (startTS, endTS, kpiItem) => tối ưu payload
    "endTS": number,
    "kpiItem": { 
        "displayName": string,
        "unit": string,
        "formula": string or null,
        "isRaw": boolean,
        "isConstant": boolean,
        "params": [
            {
                "name": string,
                "variableName": string,
                "deviceName": string,
                "deviceType": string,
                "aggregationType": string
            }
        ]
    }
}
```

- **Response**:

```json
{
    "message_en": "Parameter updated successfully",
    "message_vn": "Cập nhật biến dữ liệu thành công",
    "item": {
        "_id": "672c3a4a027ef7f660e5ae5a",
        "reportId": "672c3a39027ef7f660e5ae56",
        "orgIdName": "bia-hanoi",
        "kpiItem": {
            "displayName": "vol_SN",
            "unit": "kW",
            "formula": null,
            "isRaw": true,
            "isConstant": false,
            "params": [
                {
                    "name": "vol_SN",
                    "variableName": "",
                    "deviceName": "H 30",
                    "deviceType": "E",
                    "aggregationType": "avg",
                    "_id": "672c3ab70e4597a9fc2334b1"
                }
            ],
            "_id": "672c3ab70e4597a9fc2334b0"
        },
        "startTS": 123123123,
        "endTS": 12312392313,
        "createdAt": "2024-11-07T03:55:54.208Z",
        "updatedAt": "2024-11-07T03:57:43.275Z",
        "__v": 0
    }
}
```

#### 🗑️ DELETE /v1/pms/reports/:reportId/parameter/:id

- **Description**: Xóa parameter theo `id`.

- **Params**:
  `reportId`: id báo cáo
  `id`: id parameter

- **Response**:

```json
{
    "message_en": "Parameter deleted successfully",
    "message_vn": "Xóa biến dữ liệu thành công",
    "item": {
        "_id": "672c3a4a027ef7f660e5ae5a",
        "reportId": "672c3a39027ef7f660e5ae56",
        "orgIdName": "bia-hanoi",
        "kpiItem": {
            "displayName": "vol_SN",
            "unit": "kW",
            "formula": null,
            "isRaw": true,
            "isConstant": false,
            "params": [
                {
                    "name": "vol_SN",
                    "variableName": "",
                    "deviceName": "H 30",
                    "deviceType": "E",
                    "aggregationType": "avg",
                    "_id": "672c3ab70e4597a9fc2334b1"
                }
            ],
            "_id": "672c3ab70e4597a9fc2334b0"
        },
        "startTS": 123123123,
        "endTS": 12312392313,
        "createdAt": "2024-11-07T03:55:54.208Z",
        "updatedAt": "2024-11-07T03:57:43.275Z",
        "__v": 0
    }
}
```

#### 🔍 GET /v1/pms/reports/:reportId/parameter

- **Description**: Lấy danh sách parameter theo `reportId` của tổ chức.

- **Params**:
  `reportId`: id báo cáo
  `id`: id parameter

- **Response**:

```json
{
    "message_en": "Get parameters successfully",
    "message_vn": "Lấy danh sách biến dữ liệu thành công",
    "items": []
}
```

### Report API

API cho `Report` giúp quản lý báo cáo. Hỗ trợ các thao tác tạo mới, cập nhật, lấy thông tin và xóa.

#### ➕ POST /v1/pms/reports

- **Description**: Tạo một báo cáo.

- **Response**:

```json
{
    "message_en": "Report created successfully",
    "message_vn": "Tạo báo cáo thành công",
    "item": {
        "owner": {
            "userId": "671b0e5308b61a31e78e5412",
            "fullName": "admin",
            "email": "admin2@biahanoi.com",
            "role": "admin",
            "orgIdName": "bia-hanoi",
            "_id": "672c3a39027ef7f660e5ae57"
        },
        "reportName": "Untitled 12",
        "reportNormalizedName": "untitled 12",
        "_id": "672c3a39027ef7f660e5ae56",
        "canvasInfo": [],
        "createdAt": "2024-11-07T03:55:37.864Z",
        "updatedAt": "2024-11-07T03:55:37.864Z",
        "__v": 0
    }
}
```
  
#### 🔍 GET /v1/pms/reports/:reportId

- **Description**: Lấy thông tin báo cáo theo `reportId`.

- **Response**:

```json
{
    "message_en": "Get detail report successfully",
    "message_vn": "Lấy chi tiết báo cáo thành công",
    "items": {
        "_id": "672c24ac8b3f961f86d9ad42",
        "owner": {
            "userId": "671b0e5308b61a31e78e5412",
            "fullName": "admin",
            "email": "admin2@biahanoi.com",
            "role": "admin",
            "orgIdName": "bia-hanoi",
            "_id": "672c24ac8b3f961f86d9ad43"
        },
        "reportName": "Untitled 3",
        "reportNormalizedName": "untitled 3",
        "canvasInfo": [],
        "createdAt": "2024-11-07T02:23:40.662Z",
        "updatedAt": "2024-11-07T02:23:40.662Z",
        "__v": 0
    }
}
```

#### ✏️ PUT /v1/pms/reports/:reportId

- **Description**: Cập nhật thông tin báo cáo.

- **Params**:
  `reportId`: id báo cáo

- **Body**:

```json
{ 
    "reportName" :  string, // Có thể push tất cả hoặc đẩy thiếu (reportName, canvasInfo) => tối ưu payload
    "canvasInfo": [
        {
          "size": number,
          "id": string,
          "x": number,
          "y": number,
          "width": number,
          "height": number,
          "fill": string,
          "color": string,
          "fontStyle": string,
          "align": string,
          "name": string,

          // other
          "_id": string,
          "elementType": "parameter" or "widget" or "text"
        }
    ]
} 
```

- **Response**:

```json
{
    "message_en": "Report updated successfully",
    "message_vn": "Cập nhật báo cáo thành công",
    "item": {
        "_id": "672c1bb0a0d7337b48a178b4",
        "owner": {
            "userId": "671b0e5308b61a31e78e5412",
            "fullName": "admin",
            "email": "admin2@biahanoi.com",
            "role": "admin",
            "orgIdName": "bia-hanoi",
            "_id": "672c1bb0a0d7337b48a178b5"
        },
        "reportName": "ducanh",
        "reportNormalizedName": "ducanh",
        "canvasInfo": [
            {
                "size": 20,
                "id": "123123",
                "x": 9,
                "y": 10,
                "width": 123,
                "height": 456,
                "fill": "transparent",
                "color": "pink",
                "fontStyle": "Time new roman",
                "align": "center",
                "name": "ducanh",
                "_id": "672c214e69f1d0c19f2cd255",
                "elementType": "parameter"
            }
        ],
        "createdAt": "2024-11-07T01:45:20.209Z",
        "updatedAt": "2024-11-07T02:10:09.099Z",
        "__v": 0
    }
}
```

#### 🗑️ DELETE /v1/pms/reports/:reportId

- **Description**: Xóa báo cáo theo `id`.

- **Params**:
  `reportId`: id báo cáo
  
- **Response**:

```json
{
    "message_en": "Report deleted successfully",
    "message_vn": "Xóa báo cáo thành công",
    "item": {
        "_id": "672c1bb0a0d7337b48a178b4",
        "owner": {
            "userId": "671b0e5308b61a31e78e5412",
            "fullName": "admin",
            "email": "admin2@biahanoi.com",
            "role": "admin",
            "orgIdName": "bia-hanoi",
            "_id": "672c1bb0a0d7337b48a178b5"
        },
        "reportName": "ducanh",
        "reportNormalizedName": "ducanh",
        "canvasInfo": [
            {
                "size": 20,
                "id": "123123",
                "x": 9,
                "y": 10,
                "width": 123,
                "height": 456,
                "fill": "transparent",
                "color": "pink",
                "fontStyle": "Time new roman",
                "align": "center",
                "name": "ducanh",
                "_id": "672c214e69f1d0c19f2cd255",
                "elementType": "parameter"
            }
        ],
        "createdAt": "2024-11-07T01:45:20.209Z",
        "updatedAt": "2024-11-07T02:10:09.099Z",
        "__v": 0
    }
}
```

#### 🔍 GET /v1/pms/reports

- **Description**: Lấy danh sách báo cáo của tổ chức.

- **Query**:

```json
  pageNumber: number
  pageSize: number
  sortByName: asc or desc
  searchByName: string
```

- **Response**:

```json
{
    "message_en": "Get list report successfully",
    "message_vn": "Lấy danh sách báo cáo thành công",
    "items": [
        {
            "_id": "672c286b0ddb120c9e8c52be",
            "owner": {
                "userId": "671b0e5308b61a31e78e5412",
                "fullName": "admin",
                "email": "admin2@biahanoi.com",
                "role": "admin",
                "orgIdName": "bia-hanoi",
                "_id": "672c286b0ddb120c9e8c52bf"
            },
            "reportName": "Untitled 11",
            "reportNormalizedName": "untitled 11",
            "canvasInfo": [],
            "createdAt": "2024-11-07T02:39:39.743Z",
            "updatedAt": "2024-11-07T02:39:39.743Z",
            "__v": 0
        },
        {
            "_id": "672c2710fbef7dac51f57541",
            "owner": {
                "userId": "671b0e5308b61a31e78e5412",
                "fullName": "admin",
                "email": "admin2@biahanoi.com",
                "role": "admin",
                "orgIdName": "bia-hanoi",
                "_id": "672c2710fbef7dac51f57542"
            },
            "reportName": "Untitled 10",
            "reportNormalizedName": "untitled 10",
            "canvasInfo": [],
            "createdAt": "2024-11-07T02:33:52.534Z",
            "updatedAt": "2024-11-07T02:33:52.534Z",
            "__v": 0
        },
        {
            "_id": "672c1baaa0d7337b48a178ac",
            "owner": {
                "userId": "671b0e5308b61a31e78e5412",
                "fullName": "admin",
                "email": "admin2@biahanoi.com",
                "role": "admin",
                "orgIdName": "bia-hanoi",
                "_id": "672c1baaa0d7337b48a178ad"
            },
            "reportName": "Untitled 1",
            "reportNormalizedName": "untitled 1",
            "canvasInfo": [],
            "createdAt": "2024-11-07T01:45:14.717Z",
            "updatedAt": "2024-11-07T01:45:14.717Z",
            "__v": 0
        }
    ],
    "total": 3,
    "page": 1,
    "pageSize": 5
}
```

---

