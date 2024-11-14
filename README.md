
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
    - [➕ POST /v1/pms/reports/review](#-post-v1pmsreportsreview)
---

## Schemas

### Report Info Schema

- **Mô tả**: Lưu trữ thông tin báo cáo, bao gồm chủ sở hữu, tên báo cáo, và các phần tử trong báo cáo như `text`, `parameter` (biến dữ liệu để hiển thị giá trị khi xem lại), và `widget` (các đồ thị hoặc biểu đồ).

```javascript
const ReportInfoSchema = new Schema(
  {
    owner: { // chủ sở hữu của report
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
    reportName: { // tên report
      type: String,
      required: true,
      trim: true,
    },
    reportNormalizedName: { // tên theo dạng normailize
      type: String,
      required: true,
      trim: true,
    },
    canvasInfo: { // Các phần tử canvas trong report
      type: [
        {
          size: { type: Number, required: true }, // kích thước
          x: { type: Number, required: true }, // vị trí theo tọa đô x
          y: { type: Number, required: true }, // vị trí theo tọa độ y
          width: { type: Number, required: true }, // chiều rộng 
          height: { type: Number, required: true }, // chiều cao
          fill: { type: String, default: "transparent" }, // màu background
          color: { type: String, required: true }, // màu chữ
          fontStyle: { type: String, required: true }, // kiểu chữ time newroman
          align: { type: String, required: true }, // căn giữa
          name: { type: String, required: true }, // tên 
          uuid: { type: String, required: true }, // mã định danh phần tử cho biến paramter === object

          // other
          paramId: { type: String, required: true }, // case text không cần paramId // mã định danh cho biến parameter === class
          elementType: { type: String, required: true, enum: ["parameter", "text", "widget"] }, kiểu phần tử canvas
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
    reportId: { // mã báo cáo chứa các biến dữ liệu
      type: String,
      required: true,
      trim: true,
    },
    orgIdName: { type: String, required: true }, // tổ chức
    kpiItem: { // dữ liệu cụ thể
      type: { 
        displayName: String, // tên hiển thị
        unit: String, // đơn vị
        formula: String, // công thức
        isRaw: { // dữ liệu thô hay không 
          type: Boolean,
        },
        isConstant: { // nếu sử dụng công thức dữ liệu có ở dạng constant hay không
          type: Boolean,
          default: false,
        },
        params: [ // bộ dữ liệu để tính toán cho dữ liệu cụ thể
          {
            name: String,// tên dữ liệu 
            variableName: String,// tên biến
            deviceName: String,// tên thiết bị
            deviceType: {//loại thiết bị
              type: String,
              enum: ["E", "G", "W", "INVSOL", "SENSOL"],
            },
            aggregationType: {//cách tính toán
              type: String,
              enum: ["sum", "avg", "max", "min", "last", "counter", null],
            },
          },
        ],
      },
      required: true,
    },
    startTS: { type: Number, required: true }, // thời gian bắt đầu tính toán dữ liệu
    endTS: { type: Number, required: true }, // thời gian kết thúc
    name: { type: String }, // tên dữ liệu
    typeData: { 
      type: String, 
      enum: ["raw", "custom"]
    }, // loại dữ liệu
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
    },
    "name": string,
    "typeData": string,
}
```

- **Response**: (case success)

```json
{
    "message_en": "Parameter created successfully",
    "message_vn": "Tạo biến dữ liệu thành công",
    "item": {
        "reportId": "672ed2ca68c049c1cd146228",
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
                    "_id": "672ed38468c049c1cd14627b"
                }
            ],
            "_id": "672ed38468c049c1cd14627a"
        },
        "startTS": 123123123,
        "endTS": 12312312313,
        "name": "ducanh5",
        "typeData": "ducanh",
        "_id": "672ed38468c049c1cd146279",
        "createdAt": "2024-11-09T03:14:12.468Z",
        "updatedAt": "2024-11-09T03:14:12.468Z",
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
    "item": {
        "_id": "672e0be47398ef0a181f1351",
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
                    "_id": "672ecfa46f21d74c463d86c4"
                }
            ],
            "_id": "672ecfa46f21d74c463d86c3"
        },
        "startTS": 123123123,
        "endTS": 12312312313,
        "name": "ducanh",
        "createdAt": "2024-11-08T13:02:28.385Z",
        "updatedAt": "2024-11-09T02:57:40.092Z",
        "__v": 0,
        "typeData": "ducanh"
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
    },
    "name": string,
    "typeData": string,
}
```

- **Response**:

```json
{
    "message_en": "Parameter updated successfully",
    "message_vn": "Cập nhật biến dữ liệu thành công",
    "item": {
        "_id": "672ed38468c049c1cd146279",
        "reportId": "672ed2ca68c049c1cd146228",
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
                    "_id": "672ed3c968c049c1cd14628a"
                }
            ],
            "_id": "672ed3c968c049c1cd146289"
        },
        "startTS": 123123123,
        "endTS": 12312312313,
        "name": "ducanh5-update",
        "typeData": "ducanh",
        "createdAt": "2024-11-09T03:14:12.468Z",
        "updatedAt": "2024-11-09T03:15:21.166Z",
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
        "name": "ducanh5-update",
        "typeData": "ducanh",
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

#### 🔍 POST /v1/pms/reports/review

- **Description**: Lấy dữ liệu cho bản review báo cáo của tổ chức.

- **Query**:

```json
{
    "reportId": string,
    "canvasInfo": [
        {
            "size": number,
            "x": number,
            "y": number,
            "width": number,
            "height": number,
            "fill": string,
            "color": string,
            "fontStyle": string,
            "align": string,
            "name": string,
            "uuid": string,
            "paramId": string,
            "elementType": "parameter" // case parameter phải có paramId
        },
        {
            "size":  number,
            "x": number,
            "y": number,
            "width": number,
            "height": number,
            "fill": string,
            "color": string,
            "fontStyle": string,
            "align": string,
            "name": string,
            "uuid": string, 
            "elementType": "text" // case text không có paramId
        }
    ]
}
```

- **Response**:

```json
{
    "message_en": "Get the data of the report successfully",
    "message_vn": "Lấy dữ liệu báo cáo thành công",
    "items": [
        {
            "item": 2428249.692,
            "uuid": "report_tV8xY-1731466983329",
            "elementType": "parameter"
        },
        {
            "item": 2428249.692,
            "uuid": "report_tV8xY-1731466983329",
            "elementType": "parameter"
        }
    ]
}
```

---

