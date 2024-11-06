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
    canvasInfo: {
      parameterElements: [
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
          dateRange: {
            startTS: { type: Number, required: true },
            endTS: { type: Number, required: true },
          },
          kpiItemId: { type: String, required: true },
        },
      ],
      textElements: [
        {
          value: { type: String, required: true },
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
        },
      ],
    },
    listParameter: [
      {
        id: { type: String, require: true },
        displayName: { type: String },
        unit: { type: String },
        formula: { type: String },
        isRaw: { type: Boolean },
        isConstant: { type: Boolean, default: false },
        params: [
          {
            name: { type: String },
            variableName: { type: String },
            deviceName: { type: String },
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
    ]
  },
  { timestamps: true }
);

const ReportInfoModel = mongoose.model("ReportInfo", ReportInfoSchema);
module.exports = ReportInfoModel;

```
