# 罕见病不良反应风险预测系统 - Cloudflare Pages 部署包

## 项目说明

纯前端静态页面，基于 CatBoost ONNX 模型在浏览器中直接运行推理，无需后端服务器。

- **模型**: CatBoost (6 项最优特征)
- **推理引擎**: ONNX Runtime Web (WASM)
- **部署平台**: Cloudflare Pages / 任意静态托管

## 文件结构

```
cf-deploy/
├── index.html              # 主页面 (含全部前端逻辑)
├── model.onnx              # CatBoost ONNX 模型 (~5.2MB)
├── scaler.json             # StandardScaler 参数 (均值/标准差)
├── feature_meta.json       # 特征元数据 (范围/默认值)
├── feature_importance.json # 特征重要性 (用于近似 SHAP)
└── README.md               # 本文件
```

## 本地预览

```bash
cd cf-deploy
python -m http.server 8080
# 打开 http://localhost:8080
```

## Cloudflare Pages 部署

### 方式一：Dashboard 手动上传

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 进入 **Pages** → **创建项目**
3. 选择 **直接上传**
4. 将 `cf-deploy/` 目录下所有文件打包为 zip 上传
5. 项目构建框架选择 **None**
6. 部署完成，获得 `*.pages.dev` 域名

### 方式二：Wrangler CLI

```bash
# 安装 Wrangler
npm install -g wrangler

# 登录 Cloudflare
wrangler login

# 部署
cd cf-deploy
wrangler pages deploy .
```

### 方式三：Git 集成 (推荐)

1. 将 `cf-deploy/` 内容推送到 Git 仓库
2. 在 Cloudflare Pages 创建项目，连接该仓库
3. 构建设置：
   - **构建命令**: 留空
   - **构建输出目录**: `/` (根目录)
4. 保存并部署

## 特征列表

| 特征 | 范围 | 说明 |
|---|---|---|
| 利尿剂总剂量 | 0 ~ 260 | 用药剂量 |
| 心率HR | 19 ~ 567 | 心跳次数/分钟 |
| 钙离子(Ca2+) | 1.5 ~ 2.79 | 血清钙浓度 |
| 白细胞计数(WBC#) | 2.3 ~ 30.2 | 白细胞数量 |
| 钾离子(K+) | 2.4 ~ 7.6 | 血清钾浓度 |
| 总胆红素(TBIL) | 1.7 ~ 84 | 胆红素水平 |

## 模型性能 (测试集)

- AUC (ROC): 0.8705
- Accuracy: 0.8367
- Sensitivity: 0.6500
- Specificity: 0.8846
- Brier Score: 0.1096

## 技术栈

- HTML5 + CSS3 (无框架)
- ONNX Runtime Web 1.16.3 (CDN)
- CatBoost → ONNX 导出

## 注意事项

1. **首次加载**: model.onnx 约 5.2MB，首次加载需 2-5 秒 (CDN 缓存后加速)
2. **浏览器兼容**: 需要支持 WebAssembly 的现代浏览器 (Chrome 80+, Firefox 75+, Safari 14+)
3. **推理位置**: 所有计算在浏览器本地完成，数据不会上传到任何服务器
