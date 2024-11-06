# Schema

## Schema report
```js
const ReportInfoSchema = new Schema(
  {
    owner: {
      type: {
        userId: { type: String, required: true },
        fullName: {
          type: String,
          required: true,
        },
        email: {
          type: String,
          required: true,
        },
        role: {
          type: String,
          required: true,
          enum: ["admin", "member"],
          default: "member",
        },
        orgIdName: {
          type: String,
          required: true,
        },
      },
    },
    reportName: {
      type: String,
      required: true,
      trim: true,
    },
    reportNormailizedName: {
      type: String,
      required: true,
      trim: true,
    },
    // Các phần tử như text hay data biểu diễn dưới dạng canvas
    canvasInfo: [{
      type: String,
    }],
    // Các đồ thị
    chartItems: [{
      type: String,
    }],
  },
  { timestamps: true }
);
```

### Schema Text
![image](https://github.com/user-attachments/assets/db25f11c-01c2-4dbc-b24f-d23dba0e3c2e)

des: Các text box bên trong report
```js
const TextElementSchema = new Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    type: {
      type: String,
      required: true,
      enum: ["text"],
    },
    value: {
      type: String,
      required: true,
    },
    size: {
      type: Number,
      required: true,
    },
    id: {
      type: String,
      required: true,
      unique: true,
    },
    x: {
      type: Number,
      required: true,
    },
    y: {
      type: Number,
      required: true,
    },
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    fill: {
      type: String,
      default: "transparent", 
    },
    color: {
      type: String,
      required: true,
    },
    fontStyle: {
      type: String,
      required: true,
    },
    align: {
      type: String,
      required: true,
    },
  },
  { timestamps: true }
);
```

### Schema Data 
![image](https://github.com/user-attachments/assets/ef5a9857-c851-4434-ac52-69af6802ab5f)

des: Các text element hiển thị data bên trong report
```js
const DataElementSchema = new Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    type: {
      type: String,
      required: true,
      enum: ['data'], 
    },
    size: {
      type: Number,
      required: true,
    },
    id: {
      type: String,
      required: true,
      unique: true, 
    },
    x: {
      type: Number,
      required: true,
    },
    y: {
      type: Number,
      required: true,
    },
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    fill: {
      type: String,
      default: 'transparent',
    },
    color: {
      type: String,
      required: true,
    },
    fontStyle: {
      type: String,
      required: true,
    },
    align: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    dateRange: {
      startTS: {
        type: Number,
        required: true, 
      },
      endTS: {
        type: Number,
        required: true, 
      },
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
  },
  { timestamps: true }
);
```

### Schema DataInReport
des: Các data được định nghĩa trong report lưu tạm thời và có thể đã insert vào report, có thể chưa

![image](https://github.com/user-attachments/assets/660d9d12-cbb6-4a59-86fb-64b51b8a7043)
```js
const DataInReportSchema = new mongoose.Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    owner: {
      type: {
        userId: { type: String, required: true },
        fullName: {
          type: String,
          required: true,
        },
        email: {
          type: String,
          required: true,
        },
        role: {
          type: String,
          required: true,
          enum: ["admin", "member"],
          default: "member",
        },
        orgIdName: {
          type: String,
          required: true,
        },
      },
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
  },
  { timestamps: true }
);
```

### Schema Chart
![image](https://github.com/user-attachments/assets/82e61c8b-94a7-40b8-8c15-43dc036fc4d8)

des: Đồ thị
```js
const ChartSchema = new Schema(
  {
    reportId: {
      type: String,
      required: true,
      trim: true,
    },
    chartName: {
      type: String,
      required: true,
      trim: true,
    },
    chartNormailizedName: {
      type: String,
      required: true,
      trim: true,
    },
    typeChart: {
      type: String,
      required: true,
      enum: ["column", "line", "pie"],
    },
    kpiItems: [
      {
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
    ],
  },
  { timestamps: true }
);
```

