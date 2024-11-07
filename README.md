
# 📊 Report Management System Documentation

## 📑 Mục lục
- [📂 Schemas](#schemas)
  - [📄 Report Info Schema](#report-info-schema)
  - [📄 Parameter Schema](#parameter-schema)
- [📝 API Documentation](#api-documentation)
  - [📊 Parameter API](#parameter-api)
    - [➕ POST /v1/pms/reports/:reportId/parameter](#-post-v1pmsreportsreportidparameter)
    - [🔍 GET /v1/pms/reports/:reportId/parameter/:id](#-get-v1pmsreportsreportidparameterid)
    - [✏️ PUT /v1/pms/reports/:reportId/parameter/:id](#-put-v1pmsreportsreportidparameterid)
    - [🗑️ DELETE /v1/pms/reports/:reportId/parameter/:id](#-delete-v1pmsreportsreportidparameterid)
    - [🔍 GET /v1/pms/reports/:reportId/parameter](#-get-v1pmsreportsreportidparameter)
  - [📊 Report API](#report-api)
    - [➕ POST /api/parameters](#post-apiparameters)
    - [🔍 GET /api/parameters/:id](#get-apiparametersid)
    - [✏️ PUT /api/parameters/:id](#put-apiparametersid)
    - [🗑️ DELETE /api/parameters/:id](#delete-apiparametersid)
    - [🔍 GET /api/parameters/:id](#get-apiparametersid)

---

## Schemas

### Report Info Schema

- **Mô tả**: Lưu trữ thông tin báo cáo, bao gồm chủ sở hữu, tên báo cáo, và các phần tử trong báo cáo như `text`, `parameter` (biến dữ liệu để hiển thị giá trị khi xem lại), và `widget` (các đồ thị hoặc biểu đồ).

```javascript
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

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
    canvasInfo: [
      {
        size: { type: Number, required: true },
        id: { type: String, required: true },
        x: { type: Number, required: true },
        y: { type: Number, required: true },
        width: { type: Number, required: true },
        height: { type: Number, required: true },
        fill: { type: String, default: 'transparent' },
        color: { type: String, required: true },
        fontStyle: { type: String, required: true },
        align: { type: String, required: true },
        name: { type: String, required: true },
        uuid: { type: String, required: true },
        elementType: {
          type: String,
          required: true,
          enum: ["parameter", "text", "widget"],
        },
      },
    ],
  },
  { timestamps: true }
);

const ReportInfoModel = mongoose.model("ReportInfo", ReportInfoSchema);
module.exports = ReportInfoModel;
```

---

### Parameter Schema

- **Mô tả**: Lưu trữ biến dữ liệu của một báo cáo, giúp hiển thị danh sách dữ liệu có thể thêm vào báo cáo và cách chèn dữ liệu vào báo cáo.

```javascript
const mongoose = require("mongoose");

const ParameterSchema = new mongoose.Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    orgIdName: { type: String, required: true },
    uuid: {
      type: String,
      required: true,
    },
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
  },
  { timestamps: true }
);

const ParameterModel = mongoose.model("Parameter", ParameterSchema);
module.exports = ParameterModel;
```

---

## API Documentation

> Note: Tất cả các api cần header: { 'platform' : 'pms'} và cookies: { 'pms' : 'replaceCookie' }

### Parameter API

API cho `Parameter` giúp quản lý các biến dữ liệu của báo cáo. Hỗ trợ các thao tác tạo mới, cập nhật, lấy thông tin và xóa parameter.

#### ➕ POST /v1/pms/reports/:reportId/parameter

- **Description**: Tạo một parameter mới cho báo cáo.
  
#### 🔍 GET /v1/pms/reports/:reportId/parameter/:id

- **Description**: Lấy thông tin parameter theo `id`.

#### ✏️ PUT /v1/pms/reports/:reportId/parameter/:id 

- **Description**: Cập nhật thông tin parameter.

#### 🗑️ DELETE /v1/pms/reports/:reportId/parameter/:id

- **Description**: Xóa parameter theo `id`.

#### 🔍 GET /v1/pms/reports/:reportId/parameter

- **Description**: Lấy danh sách parameter theo `reportId` của tổ chức.

### Report API

API cho `Report` giúp quản lý báo cáo. Hỗ trợ các thao tác tạo mới, cập nhật, lấy thông tin và xóa.

#### ➕ POST /v1/pms/reports

- **Description**: Tạo một báo cáo.
  
#### 🔍 GET /v1/pms/reports/:reportId

- **Description**: Lấy thông tin báo cáo theo `reportId`.

#### ✏️ PUT /v1/pms/reports/:reportId

- **Description**: Cập nhật thông tin báo cáo.

#### 🗑️ DELETE /v1/pms/reports/:reportId

- **Description**: Xóa báo cáo theo `id`.

#### 🔍 GET /v1/pms/reports

- **Description**: Lấy danh sách báo cáo của tổ chức.

---

## Usage Examples

