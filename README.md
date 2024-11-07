
# Report Management System Documentation

## Mục lục
- [Schemas](#schemas)
  - [Report Info Schema](#report-info-schema)
  - [Parameter Schema](#parameter-schema)
- [API Documentation](#api-documentation)
  - [Parameter API](#parameter-api)
    - [POST /api/parameters](#post-apiparameters)
    - [GET /api/parameters/:id](#get-apiparametersid)
    - [PUT /api/parameters/:id](#put-apiparametersid)
    - [DELETE /api/parameters/:id](#delete-apiparametersid)
- [Usage Examples](#usage-examples)

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

### Parameter API

API cho `Parameter` giúp quản lý các biến dữ liệu của báo cáo. Hỗ trợ các thao tác tạo mới, cập nhật, lấy thông tin và xóa parameter.

#### POST /api/parameters

- **Description**: Tạo một parameter mới cho báo cáo.
- **Body Parameters**:
  - `reportId` (string, required): ID của báo cáo.
  - `orgIdName` (string, required): Tên của tổ chức.
  - `uuid` (string, required): UUID duy nhất của parameter.
  - `kpiItem` (object, required): Thông tin chi tiết về KPI.
  - `startTS` (number, required): Timestamp bắt đầu.
  - `endTS` (number, required): Timestamp kết thúc.
- **Response**: Parameter mới được tạo.

#### GET /api/parameters/:id

- **Description**: Lấy thông tin parameter theo `id`.
- **Response**: Chi tiết của parameter với `id` tương ứng.

#### PUT /api/parameters/:id

- **Description**: Cập nhật thông tin parameter.
- **Body Parameters**: Tương tự `POST /api/parameters`, các giá trị cần cập nhật.
- **Response**: Parameter sau khi cập nhật.

#### DELETE /api/parameters/:id

- **Description**: Xóa parameter theo `id`.
- **Response**: Trạng thái xóa parameter.

---

## Usage Examples

- **Tạo mới một parameter**:
  ```json
  POST /api/parameters
  {
    "reportId": "123",
    "orgIdName": "ABC Organization",
    "uuid": "unique-uuid-456",
    "kpiItem": {
      "displayName": "Sales KPI",
      "unit": "%",
      "formula": "sum(sales)/target",
      "isRaw": false,
      "isConstant": true,
      "params": [
        {
          "name": "Sales",
          "variableName": "sales",
          "deviceName": "POS Device",
          "deviceType": "E",
          "aggregationType": "sum"
        }
      ]
    },
    "startTS": 1672500000,
    "endTS": 1672600000
  }
  ```

--- 
