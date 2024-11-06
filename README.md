### Schema report

```js
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
    canvasInfo: [{
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

          // other
          uuid: { type: String, required: true },
          elementType: { type: String, required: true, enum: ["parameter","text","widget"] },
      }],
  },
  { timestamps: true }
);

const ReportInfoModel = mongoose.model("ReportInfo", ReportInfoSchema);
module.exports = ReportInfoModel;

```

### Schema parameter

```js
const mongoose = require("mongoose");

// Define the schema
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
