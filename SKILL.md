---
name: lginnotek-fiori-common
description: |
  LGInnotek SAP Fiori 프로젝트 공통 개발 가이드. Controller, Fragment 등 새 파일 생성 시 반드시 적용해야 하는 공통 규칙을 정의합니다.
  Use this skill when the user asks to create a new List Report Extension Controller, Fragment, or any Fiori screen file in an LGInnotek SAP Fiori project.
metadata:
  version: "1.0.0"
---

# LGInnotek SAP Fiori 프로젝트 공통 개발 가이드

> 이 파일은 SAP Fiori 화면 개발 시 공통으로 적용해야 하는 작업을 정의합니다.
> 새 화면(Controller, Fragment 등) 생성 시 반드시 아래 규칙을 따르세요.

---

## ⚠️ AI 작업 필수 준수 규칙

> **이 규칙은 모든 작업에서 가장 먼저, 가장 우선적으로 적용됩니다.**

### 규칙 1: SKILL.md 파일은 절대 수정 금지

- AI는 이 SKILL.md 파일을 **절대 수정하지 않는다.**
- 소스 파일(Controller, Fragment, manifest.json 등)만 이 SKILL.md를 기준으로 작성/수정한다.
- 사용자가 명시적으로 "SKILL.md를 수정해줘"라고 요청한 경우에만 수정한다.

### 규칙 2: SKILL.md 내 코드는 반드시 터미널 원문으로 확인 후 사용

- AI가 SKILL.md를 읽을 때 시스템이 일부 식별자(변수명, 파라미터명 등)를 **자동으로 마스킹** 처리한다.
- 따라서 함수 호출의 파라미터명, 상수명 등 코드 관련 내용은 **반드시 아래 방법으로 터미널 원문을 확인한 후 사용**한다.

```python
# 반드시 이 방법으로 원문 확인
python3 << 'EOF'
with open('/home/user/.copilot/skills/lginnotek-fiori-common/SKILL.md', 'r') as f:
    content = f.read()
idx = content.find('확인할_함수명')
end = content.find(')', idx) + 1
print(repr(content[idx:end]))
EOF
```

- 특히 `wireOnce`, `v2AnalyticalTablewireOnce` 등 **함수 호출의 파라미터 목록**은 반드시 터미널로 확인한다.
- 터미널 확인 없이 마스킹된 값을 그대로 소스에 사용하는 것을 금지한다.

### 규칙 3: Case B (GridTable) wireOnce 파라미터 확인 완료

> 터미널 원문 확인으로 검증된 실제 파라미터 (마스킹 우회 확인값)

```javascript
// Case B: GridTable (V4) - wireOnce 실제 파라미터 4개
GridTableRowCount.wireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_INFO_ID, FILTERBAR_ID);
```

| 순서 | 파라미터명 | 대응 상수 |
|:---:|---|---|
| 1 | `oView` | `this.getView()` |
| 2 | `TBL_ID` | `{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable` |
| 3 | `TITLE_ID` | `{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title` |
| 4 | `FILTERBAR_INFO_ID` | `{AppId}::{EntityList}--fe::table::{Entity}::LineItem-filterInfoBar` |
| 5 | `FILTERBAR_ID` | `{AppId}::{EntityList}--fe::table::{Entity}::FilterBar` |

> ⚠️ **`FILTERBAR_INFO_ID` 값 주의사항 (검증된 표준 패턴)**
>
> - **반드시 `LineItem-filterInfoBar` (LineItem- prefix + 소문자 `f`) 로 작성**한다.
> - `::FilterInfoBar` (대문자 F, LineItem prefix 없음) 형태는 **구버전 오류 패턴**이므로 사용하지 않는다.
> - 라이브러리 `ext/lib/common/GridTableRowCount.js` 터미널 시그니처: `wireOnce(view, TBL_ID, TITLE_ID,FILTER_INFOBAR, FILTERBAR_ID)` — 4번째 파라미터로 `view.byId(FILTER_INFOBAR)` 를 실행하므로 view 상 실제 존재하는 `LineItem-filterInfoBar` ID를 사용해야 한다.
> - 검증 프로젝트 (모두 `LineItem-filterInfoBar` 사용):
>   - `zpmdc04sumaregen`, `zpmdc04sumarecom`, `zpmdc04sumareset`, `zpmdc04sumarewit`, `zpmdc04sumarebad`
>   - `zesdc02_salesteamreport`, `zesdc02_salesteamlist` (Object Page), `zesdc04_salescl`, `zffic10_hol_bonus`, `zesdc06_lccodeinfo`
> - Object Page 자식 테이블(Navigation Path 사용)의 경우에도 동일 규칙:
>   `{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath}::LineItem-filterInfoBar`

---

## 0. manifest.json 자동 분석 규칙 (필수 선행 작업)

> ⚠️ **Controller, Fragment 등 새 파일 생성 전에 반드시 `webapp/manifest.json`을 읽어서 아래 값을 자동으로 추출해야 합니다.**

### 추출 경로 및 방법

| 변수 | manifest.json 추출 경로 | 추출 방법 |
|---|---|---|
| `{AppId}` | `sap.app.id` | 문자열 그대로 사용 |
| `{AppTitle}` | `sap.app.title` (i18n 키면 `i18n/i18n.properties`에서 실제값 조회) | 문자열 그대로 사용 |
| `{OData서비스명}` | `sap.app.dataSources.mainService.uri` | URI에서 서비스명 추출 (예: `/sap/opu/odata4/sap/{서비스명}/...`) |
| `{EntityList}` | `sap.ui5.routing.targets` 의 List Report target `id` | `"id": "ZPMDC01_SALESBOMHEADER_CList"` → `ZPMDC01_SALESBOMHEADER_CList` |
| `{Entity}` | `sap.ui5.routing.targets.[ListTarget].options.settings.contextPath` | `/ZPMDC01_SALESBOMHEADER_C` → 앞의 `/` 제거 → `ZPMDC01_SALESBOMHEADER_C` |
| `{TableType}` | `sap.ui5.routing.targets.[ListTarget].options.settings.controlConfiguration["@com.sap.vocabularies.UI.v1.LineItem"].tableSettings.type` | `ResponsiveTable` / `GridTable` / `AnalyticalTable` |
| `{ODataVersion}` | `sap.app.dataSources.mainService.settings.odataVersion` | `4.0` → V4, `2.0` → V2 |

### 현재 프로젝트 추출 결과 예시

```
manifest.json 분석 결과:
- AppId        : zpmdc01salbomcreate                    ← sap.app.id
- AppTitle     : Sales BOM Header Create                ← i18n 조회
- OData 서비스  : zpmdc01_salbomcreate_ui_v4             ← dataSources.mainService.uri
- OData 버전   : V4 (4.0)                               ← odataVersion
- EntityList   : ZPMDC01_SALESBOMHEADER_CList           ← routing.targets[0].id
- Entity       : ZPMDC01_SALESBOMHEADER_C               ← contextPath에서 / 제거
- TableType    : ResponsiveTable                        ← tableSettings.type
```

### 동작 규칙

> ⚠️ **"새 List Report Extension Controller 만들어줘" 등 Controller/Fragment 생성 요청 시 아래 순서를 반드시 준수한다.**

#### 전체 실행 순서 (순서 변경 금지)

| 순서 | 작업 | 설명 |
|:---:|---|---|
| **1** | **`webapp/manifest.json` 읽기** | AppId, Entity, TableType 등 값 추출 |
| **2** | **`Jenkinsfile` 생성** | 프로젝트 루트에 `/Jenkinsfile` 없으면 반드시 먼저 생성 (섹션 1 참조) |
| **3** | **`Component.js` 수정** | 첫째 줄에 `jQuery.sap.registerModulePath` 추가 (섹션 2-1 참조) |
| **4** | **`manifest.json` 수정** | `sap.f`, `ext.lib.common` lib 추가 + `variantManagement` → `"Control"` (섹션 2-2, 2-3 참조) |
| **5** | **Controller 생성/수정** | 헤더 + Variant + RowCount 포함 (섹션 3, 4 참조) |
| **6** | **ObjectPage Controller 생성/수정** | Object Page에 자식 테이블(LineItem Facet)이 있으면 RowCount 적용 (섹션 3-4 참조) |

#### 세부 규칙

1. **`webapp/manifest.json`을 읽어** `{AppId}`, `{EntityList}`, `{Entity}` 등을 **실제 값으로 치환**한다.
2. **`Jenkinsfile`이 프로젝트 루트에 없으면 가장 먼저 생성**한다. 이미 존재하면 건너뛴다.
3. 추출한 `{TableType}`과 `{ODataVersion}`을 기준으로 **RowCount 처리 방식(Case A/B/C)을 자동 선택**한다.
4. 모든 ID 상수, byId() 호출, ControllerExtension.extend() 경로에 실제 값을 적용한다.
5. **Controller 파일 존재 여부에 따라 아래와 같이 처리한다.**
   - **파일이 없으면 → 신규 생성**: 섹션 5-1의 전체 템플릿 코드를 그대로 작성한다.
   - **파일이 이미 있으면 → 기존 파일 수정**: 섹션 5-2의 체크리스트를 따라 누락 항목만 추가/수정한다.

> ⚠️ **"기존 파일 수정" 시 주의사항**
> - 기존 `onInit` 등 커스텀 코드는 절대 삭제하지 않는다.
> - 파일 헤더가 없으면 반드시 파일 최상단에 추가한다.
> - `sap.ui.define` 모듈 배열에 `Variant`, RowCount 라이브러리가 없으면 추가한다.
> - 상수 ID 블록(`TBL_ID`, `TITLE_ID`, `FILTERBAR_ID` 등)이 없으면 `sap.ui.define` 콜백 함수 첫 줄에 추가한다.
> - `downloadPDF` 함수가 없으면 상수 블록 바로 아래에 추가한다.
> - `onAfterRendering`이 없으면 `override` 블록 내 `onInit` 뒤에 추가한다.
> - `onAfterRendering`이 이미 있으면 기존 내용을 유지하면서 Variant 초기화 및 setTimeout RowCount 코드만 내부에 추가한다.

---

## 1. Jenkinsfile 생성

> **Controller/Fragment 생성 요청 시 가장 먼저 실행한다.**
> 프로젝트 루트(`/Jenkinsfile`)에 파일이 없으면 즉시 생성하고, 이미 존재하면 건너뛴다.

- **위치**: 프로젝트 root 직하 (`/Jenkinsfile`)
- **내용**: 아래 내용을 그대로 사용합니다.

```groovy
// 1. GUI 등록 없이, 코드에서 직접 Git 주소를 적어서 라이브러리를 당겨옵니다.
library identifier: 'devops-lib@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://gitlab.aws.lginnotek.com/erp/cicd_pipelines.git',
   credentialsId: 'd13a9a02-3811-4120-bdf4-c1c290e0d1ed']
)

pipeline_fiori()
```

---

## 2. 공통 Lib 등록 처리

> ⚠️ **아래 2-1 ~ 2-4 각 항목은 수정 전 반드시 파일을 읽어 이미 적용되어 있는지 확인한다.**
> 이미 적용된 항목은 건너뛰고, 누락된 항목만 추가/수정한다.

### 2-1. `Component.js` - 첫째 줄에 Lib 경로 등록

**확인 방법**: `Component.js` 파일을 읽어 첫째 줄에 `jQuery.sap.registerModulePath` 가 있는지 확인한다.
- **없으면** → `sap.ui.define(` 바로 위 첫째 줄에 추가한다.
- **이미 있으면** → 건너뛴다.

```javascript
jQuery.sap.registerModulePath("ext.lib.common", "/sap/bc/ui5_ui5/sap/zlibrary");
sap.ui.define(
    ["sap/fe/core/AppComponent"],
    function (Component) {
        "use strict";
        return Component.extend("{AppId}.Component", {
            metadata: { manifest: "json" }
        });
    }
);
```

> ✅ 실제 적용 예시 (`zpmdc01salbomcreate/webapp/Component.js`):
> ```javascript
> jQuery.sap.registerModulePath("ext.lib.common", "/sap/bc/ui5_ui5/sap/zlibrary");
> sap.ui.define(["sap/fe/core/AppComponent"], function (Component) { ... });
> ```

---

### 2-2. `manifest.json` - `sap.ui5 > dependencies > libs`에 Lib 등록

**확인 방법**: `manifest.json`의 `sap.ui5.dependencies.libs`를 읽어 각 항목 존재 여부를 확인한다.
- `"sap.f": {}` → **없으면** 추가, 이미 있으면 건너뛴다.
- `"ext.lib.common": {}` → **없으면** 추가, 이미 있으면 건너뛴다.

```json
"sap.ui5": {
  "dependencies": {
    "libs": {
      "sap.m": {},
      "sap.ui.core": {},
      "sap.fe.templates": {},
      "sap.f": {},
      "ext.lib.common": {}
    }
  }
}
```

---

### 2-3. `manifest.json` - List Report target의 `variantManagement`를 `Control`로 설정

**확인 방법**: `sap.ui5.routing.targets.{EntityList}.options.settings.variantManagement` 값을 확인한다.
- 값이 `"Page"` 또는 없으면 → **`"Control"`로 변경**한다.
- 이미 `"Control"`이면 → 건너뛴다.

```json
"targets": {
  "{EntityList}": {
    "type": "Component",
    "id": "{EntityList}",
    "name": "sap.fe.templates.ListReport",
    "options": {
      "settings": {
        "contextPath": "/{Entity}",
        "variantManagement": "Control",
        ...
      }
    }
  }
}
```

> - 기본값 `"Page"` → **`"Control"`** 로 변경
> - `Variant.headerVariant()` / `Variant.tableVariant()` 호출과 함께 동작하기 위해 필수 설정

---

### 2-4. `manifest.json` - Controller Extension 등록 확인

**확인 방법**: `sap.ui5.extends.extensions.sap.ui.controllerExtensions` 블록이 있는지 확인한다.
- **없으면** → 아래 블록을 `sap.ui5` 내에 추가한다.
- **이미 있으면** → `controllerName` 경로가 `{AppId}.ext.controller.ListReportPageExt` 인지 확인하고 다르면 수정한다.

```json
"sap.ui5": {
  ...
  "extends": {
    "extensions": {
      "sap.ui.controllerExtensions": {
        "sap.fe.templates.ListReport.ListReportController": {
          "controllerName": "{AppId}.ext.controller.ListReportPageExt"
        }
      }
    }
  }
}
```

> ✅ 실제 적용 예시:
> ```json
> "extends": {
>   "extensions": {
>     "sap.ui.controllerExtensions": {
>       "sap.fe.templates.ListReport.ListReportController": {
>         "controllerName": "zffic30verifycrettest5.ext.controller.ListReportPageExt"
>       }
>     }
>   }
> }
> ```
>
> ⚠️ 이 블록이 없으면 Controller 파일이 있어도 Fiori Elements가 인식하지 못해 동작하지 않는다.

---

### 2-5. `manifest.json` - Object Page 존재 시 Flexible Column Layout 설정 (필수)

**확인 방법**: `sap.ui5.routing.targets` 에 `sap.fe.templates.ObjectPage` 타입의 target이 있는지 확인한다.
- Object Page target이 있으면 → 아래 FCL 설정이 모두 적용되어 있는지 확인하고, 없거나 다른 항목만 추가/수정한다.
- Object Page target이 없으면 → 이 섹션 전체를 건너뛴다.

#### FCL 적용 항목 (Object Page 존재 시 전부 필수)

**① `sap.ui5.routing.config`에 FCL 라우터 및 레이아웃 설정 추가**

```json
"sap.ui5": {
  "routing": {
    "config": {
      "routerClass": "sap.f.routing.Router",
      "flexibleColumnLayout": {
        "defaultTwoColumnLayoutType": "TwoColumnsMidExpanded",
        "defaultThreeColumnLayoutType": "ThreeColumnsEndExpanded"
      }
    }
  }
}
```

**② List Report target에 `controlAggregation` 및 `contextPattern` 추가**

```json
"targets": {
  "{EntityList}": {
    ...
    "controlAggregation": "beginColumnPages",
    "contextPattern": ""
  }
}
```

**③ Object Page target에 `controlAggregation` 및 `contextPattern` 추가**

```json
"targets": {
  "{ObjectPageTarget}": {
    ...
    "controlAggregation": "midColumnPages",
    "contextPattern": "/{Entity}({key})"
  }
}
```

> ✅ `contextPattern` 값은 Object Page route의 `pattern` 에서 `:?query:` 를 제거한 형태로 작성한다.
> 예: pattern이 `"ZFFIC30_VERFYCR_U({key}):?query:"` 이면 contextPattern은 `"/ZFFIC30_VERFYCR_U({key})"` 이다.

**④ Object Page route에 List target도 함께 포함 (두 컬럼 동시 표시)**

```json
"routes": [
  {
    "pattern": ":?query:",
    "name": "{EntityList}Route",
    "target": ["{EntityList}"]
  },
  {
    "pattern": "{Entity}({key}):?query:",
    "name": "{ObjectPageTarget}Route",
    "target": [
      "{EntityList}",
      "{ObjectPageTarget}"
    ]
  }
]
```

**⑤ `sap.ui5.rootView`를 FCL RootContainer View로 변경**

```json
"sap.ui5": {
  "rootView": {
    "viewName": "sap.fe.templates.RootContainer.view.Fcl",
    "type": "XML",
    "async": true,
    "id": "appRootView"
  }
}
```

> ✅ 실제 적용 예시 (`manifest.json` 전체 FCL 관련 부분):
> ```json
> "rootView": {
>   "viewName": "sap.fe.templates.RootContainer.view.Fcl",
>   "type": "XML",
>   "async": true,
>   "id": "appRootView"
> },
> "routing": {
>   "config": {
>     "routerClass": "sap.f.routing.Router",
>     "flexibleColumnLayout": {
>       "defaultTwoColumnLayoutType": "TwoColumnsMidExpanded",
>       "defaultThreeColumnLayoutType": "ThreeColumnsEndExpanded"
>     }
>   },
>   "routes": [
>     {
>       "pattern": ":?query:",
>       "name": "ZFFIC30_VERFYCR_UList",
>       "target": ["ZFFIC30_VERFYCR_UList"]
>     },
>     {
>       "pattern": "ZFFIC30_VERFYCR_U({key}):?query:",
>       "name": "ZFFIC30_VERFYCR_UObjectPage",
>       "target": ["ZFFIC30_VERFYCR_UList", "ZFFIC30_VERFYCR_UObjectPage"]
>     }
>   ],
>   "targets": {
>     "ZFFIC30_VERFYCR_UList": {
>       ...
>       "controlAggregation": "beginColumnPages",
>       "contextPattern": ""
>     },
>     "ZFFIC30_VERFYCR_UObjectPage": {
>       ...
>       "controlAggregation": "midColumnPages",
>       "contextPattern": "/ZFFIC30_VERFYCR_U({key})"
>     }
>   }
> }
> ```

---

### manifest.json 체크리스트 요약

| # | 확인 항목 | 경로 | 없거나 다를 때 처리 |
|:---:|---|---|---|
| 1 | `jQuery.sap.registerModulePath` | `Component.js` 첫째 줄 | 추가 |
| 2 | `"sap.f": {}` | `sap.ui5.dependencies.libs` | 추가 |
| 3 | `"ext.lib.common": {}` | `sap.ui5.dependencies.libs` | 추가 |
| 4 | `"variantManagement": "Control"` | `sap.ui5.routing.targets.{EntityList}.options.settings` | `"Control"`로 변경 |
| 5 | `sap.ui.controllerExtensions` 블록 | `sap.ui5.extends.extensions` | 블록 전체 추가 |
| 6 | **FCL 설정 전체** (Object Page 있을 때만) | `sap.ui5.routing`, `sap.ui5.rootView` | 섹션 2-5 참조하여 5개 항목 적용 |

---

## 3. 화면 공통 변경사항 (Controller)

### 3-1. VariantManagement 위치 이동

- 상단 Title 및 Table 영역의 VariantManagement 위치 이동 처리
- `Variant.headerVariant()` 및 `Variant.tableVariant()` 호출로 처리

---

### 3-2. PDF 메뉴얼 버튼 연결 (Variant 포함)

#### `sap.ui.define`에 추가
```javascript
sap.ui.define([
  'sap/ui/core/mvc/ControllerExtension',
  "ext/lib/common/Variant",          // ← 추가
  ...
], function (ControllerExtension, Variant, ...) {
```

#### Controller 내 `downloadPDF` 함수 추가 (상수 정의부 아래)
```javascript
function downloadPDF() {
    const pdfUrl = "https://lginnotek.sharepoint.com/..."; // 실제 메뉴얼 URL로 변경
    window.open(pdfUrl, "_blank");
}
```

#### `onAfterRendering`에 Variant 초기화 추가
```javascript
onAfterRendering: function () {
    const oView    = this.getView();
    const bManual  = false;
    const oTableVM = oView.byId("{AppId}::{EntityList}--fe::table::{Entity}::LineItem::VM");
    Variant.headerVariant(this, bManual, downloadPDF);
    Variant.tableVariant(oTableVM);
    ...
}
```

> ✅ 실제 적용 예시 (`ListReportPageExt.controller.js`):
> ```javascript
> const oTableVM = oView.byId("zpmdc01salbomcreate::ZPMDC01_SALESBOMHEADER_CList--fe::table::ZPMDC01_SALESBOMHEADER_C::LineItem::VM");
> Variant.headerVariant(this, bManual, downloadPDF);
> Variant.tableVariant(oTableVM);
> ```

---

### 3-3. Table Row Count 처리

테이블 유형에 따라 아래 3가지 중 하나를 적용합니다.

---

#### ✅ Case A: ResponsiveTable (V4 서비스)

**`sap.ui.define`에 추가**
```javascript
"ext/lib/common/ResponsiveTableRowCount"
```

**상수 ID 정의**
```javascript
const TABLE_ID     = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable";
const TITLE_ID     = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title";
const FILTERBAR_ID = "{AppId}::{EntityList}--fe::table::{Entity}::FilterBar";
```

**`onAfterRendering`의 `setTimeout` 내부에 추가**
```javascript
setTimeout(() => {
    ResponsiveTableRowCount.wireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_ID);
}, 1000);
```

> ✅ 실제 적용 예시:
> ```javascript
> const TABLE_ID     = "zpmdc01salbomcreate::ZPMDC01_SALESBOMHEADER_CList--fe::table::ZPMDC01_SALESBOMHEADER_C::LineItem-innerTable";
> const TITLE_ID     = "zpmdc01salbomcreate::ZPMDC01_SALESBOMHEADER_CList--fe::table::ZPMDC01_SALESBOMHEADER_C::LineItem-title";
> const FILTERBAR_ID = "zpmdc01salbomcreate::ZPMDC01_SALESBOMHEADER_CList--fe::table::ZPMDC01_SALESBOMHEADER_C::FilterBar";
> ...
> setTimeout(() => {
>     ResponsiveTableRowCount.wireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_ID);
> }, 1000);
> ```

---

#### Case B: GridTable (V4 서비스)

**`sap.ui.define`에 추가**
```javascript
"ext/lib/common/GridTableRowCount"
```

**상수 ID 정의** (FILTERBAR_INFO_ID 추가)
```javascript
const TBL_ID          = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable";
const TITLE_ID          = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title";
const FILTERBAR_ID      = "{AppId}::{EntityList}--fe::table::{Entity}::FilterBar";
const FILTERBAR_INFO_ID = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-filterInfoBar";
```

> ⚠️ **`FILTERBAR_INFO_ID` 값은 반드시 `LineItem-filterInfoBar` (소문자 `f`, `LineItem-` prefix 포함) 로 작성한다.**
> `::FilterInfoBar` (대문자 F, prefix 없음) 는 구버전 오류 패턴이므로 사용 금지. 자세한 근거는 상단 "규칙 3" 참조.

**`onAfterRendering`의 `setTimeout` 내부에 추가** (파라미터 5개)
```javascript
setTimeout(() => {
    GridTableRowCount.wireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_INFO_ID, FILTERBAR_ID);
}, 1000);
```

> ✅ 실제 적용 예시 (`zpmdc04sumaregen/webapp/ext/controller/ListReportPageExt.controller.js`):
> ```javascript
> const TABLE_ID          = "zpmdc04sumaregen::ZPMDC04_SUPPLIER_HEADER_R_CList--fe::table::ZPMDC04_SUPPLIER_HEADER_R_C::LineItem-innerTable";
> const TITLE_ID          = "zpmdc04sumaregen::ZPMDC04_SUPPLIER_HEADER_R_CList--fe::table::ZPMDC04_SUPPLIER_HEADER_R_C::LineItem-title";
> const FILTERBAR_ID      = "zpmdc04sumaregen::ZPMDC04_SUPPLIER_HEADER_R_CList--fe::table::ZPMDC04_SUPPLIER_HEADER_R_C::FilterBar";
> const FILTERBAR_INFO_ID = "zpmdc04sumaregen::ZPMDC04_SUPPLIER_HEADER_R_CList--fe::table::ZPMDC04_SUPPLIER_HEADER_R_C::LineItem-filterInfoBar";
> ...
> setTimeout(() => {
>     GridTableRowCount.wireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_INFO_ID, FILTERBAR_ID);
> }, 1000);
> ```

---

#### Case C: AnalyticalTable (V2 서비스)

**`sap.ui.define`에 추가**
```javascript
"ext/lib/common/AnalyticalTableRowCount"
```

**상수 ID 정의**
```javascript
const TBL_ID       = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable";
const TITLE_ID     = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title";
const FILTERBAR_ID = "{AppId}::{EntityList}--fe::table::{Entity}::FilterBar";
```

**`onAfterRendering`의 `setTimeout` 내부에 추가**
```javascript
setTimeout(() => {
    AnalyticalTableRowCount.v2AnalyticalTablewireOnce(oView, TBL_ID, TITLE_ID, FILTERBAR_ID);
}, 1000);
```

---

### 3-4. Object Page 자식 테이블 Row Count 처리

> ⚠️ **이 섹션은 Object Page(`ObjectPageExt.controller.js`)에 자식 테이블(LineItem Facet)이 있을 때만 적용한다.**
> Object Page에는 FilterBar가 없으므로 **테이블 툴바(`-toolbar`)** 를 4번째 파라미터로 전달한다.

#### 적용 대상 확인 방법

1. `manifest.json`에서 `sap.fe.templates.ObjectPage` 타입 target이 있는지 확인한다.
2. `metadata.xml` 또는 `annotation.xml`에서 Object Page 엔티티의 `SAP__UI.Facets`를 확인한다.
3. Facet 중 `Target AnnotationPath="{NavigationPath}/@SAP__UI.LineItem"` 형태의 `ReferenceFacet`이 있으면 → **해당 `{NavigationPath}`가 자식 테이블**이다.

#### Object Page 테이블 ID 명명 규칙 (검증 완료)

> ✅ **List Report와 달리 Object Page 테이블은 Facet ID가 아닌 Navigation Path(소문자, `_`로 시작)를 세그먼트로 사용한다.**

| 종류 | ID 패턴 |
|---|---|
| Table (innerTable) | `{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath}::LineItem-innerTable` |
| Title | `{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath}::LineItem-title` |
| Toolbar (FilterBar 대체) | `{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath}::LineItem-toolbar` |

> - `{ObjectPageTarget}`: `manifest.json` → Object Page target `id` (예: `ZPMDC04_SUPPLIER_HEADER_CObjectPage`)
> - `{NavigationPath}`: metadata의 `AnnotationPath="{NavigationPath}/@SAP__UI.LineItem"`에서 추출 (예: `_supcomcode`, `_suppurch`)

#### `sap.ui.define`에 추가

```javascript
"ext/lib/common/ResponsiveTableRowCount"
```

#### 상수 ID 정의 (테이블 개수만큼 반복)

```javascript
// Navigation Path: {NavigationPath1} (예: _supcomcode)
const {NAV1}_TABLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-innerTable";
const {NAV1}_TITLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-title";
const {NAV1}_FILTERBAR_ID = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-toolbar";

// Navigation Path: {NavigationPath2} (예: _suppurch)
const {NAV2}_TABLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-innerTable";
const {NAV2}_TITLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-title";
const {NAV2}_FILTERBAR_ID = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-toolbar";
```

#### `onAfterRendering`의 `setTimeout` 내부에 추가 (테이블 개수만큼 반복 호출)

```javascript
onAfterRendering: function () {
    const oView = this.getView();

    // Object Page 자식 테이블별 Row Count 이벤트 연결
    // Object Page 에는 FilterBar 가 없으므로 4번째 파라미터는 툴바 ID(-toolbar) 전달
    setTimeout(() => {
        ResponsiveTableRowCount.wireOnce(oView, {NAV1}_TABLE_ID, {NAV1}_TITLE_ID, {NAV1}_FILTERBAR_ID);
        ResponsiveTableRowCount.wireOnce(oView, {NAV2}_TABLE_ID, {NAV2}_TITLE_ID, {NAV2}_FILTERBAR_ID);
    }, 1000);
}
```

#### 전체 템플릿 (`ObjectPageExt.controller.js`)

```javascript
/*
*******************************************************************
1. Module (Sub) : {모듈명}
2. App Id       : {AppId}
3. App Title    : {AppTitle}
4. File Name    : ObjectPageExt.controller.js
5. OData        : {OData서비스명} ({ODataVersion})
6. Page Type    : Fiori Elements (Object Page Extension)
7. Description  : {AppTitle} Object Page Extension Controller
*******************************************************************
Modification LOG
-------------------------------------------------------------------
No.   CSR/PJT No.        Date        Author     Description
----  ----------------  ----------  ---------  ------------------
001                     {YYYY/MM/DD}  {사번}     Initial Release
*******************************************************************
Comment:

*******************************************************************
*/
sap.ui.define([
    'sap/ui/core/mvc/ControllerExtension',
    "ext/lib/common/ResponsiveTableRowCount"
], function (ControllerExtension, ResponsiveTableRowCount) {
    'use strict';

    // Object Page 자식 테이블 ID 상수 (테이블 개수만큼 반복 정의)
    const {NAV1}_TABLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-innerTable";
    const {NAV1}_TITLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-title";
    const {NAV1}_FILTERBAR_ID = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath1}::LineItem-toolbar";

    const {NAV2}_TABLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-innerTable";
    const {NAV2}_TITLE_ID     = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-title";
    const {NAV2}_FILTERBAR_ID = "{AppId}::{ObjectPageTarget}--fe::table::{NavigationPath2}::LineItem-toolbar";

    return ControllerExtension.extend('{AppId}.ext.controller.ObjectPageExt', {
        override: {
            onInit: function () {
                var oModel = this.base.getExtensionAPI().getModel();
            },

            onAfterRendering: function () {
                const oView = this.getView();

                setTimeout(() => {
                    ResponsiveTableRowCount.wireOnce(oView, {NAV1}_TABLE_ID, {NAV1}_TITLE_ID, {NAV1}_FILTERBAR_ID);
                    ResponsiveTableRowCount.wireOnce(oView, {NAV2}_TABLE_ID, {NAV2}_TITLE_ID, {NAV2}_FILTERBAR_ID);
                }, 1000);
            }
        }
    });
});
```

#### ✅ 실제 적용 예시 (`zpmdc04displaysup/webapp/ext/controller/ObjectPageExt.controller.js`)

**metadata.xml Facet 정의:**
```xml
<Record Type="SAP__UI.ReferenceFacet">
    <PropertyValue Property="Label" String="Company Data"/>
    <PropertyValue Property="ID" String="SUPCOMPANY"/>
    <PropertyValue Property="Target" AnnotationPath="_supcomcode/@SAP__UI.LineItem"/>  <!-- ← NavigationPath -->
</Record>
<Record Type="SAP__UI.ReferenceFacet">
    <PropertyValue Property="Label" String="Purchasing Data"/>
    <PropertyValue Property="ID" String="SUPPURCH"/>
    <PropertyValue Property="Target" AnnotationPath="_suppurch/@SAP__UI.LineItem"/>    <!-- ← NavigationPath -->
</Record>
```

**Controller 적용 결과:**
```javascript
const COMPANY_TABLE_ID     = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_supcomcode::LineItem-innerTable";
const COMPANY_TITLE_ID     = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_supcomcode::LineItem-title";
const COMPANY_FILTERBAR_ID = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_supcomcode::LineItem-toolbar";

const PURCH_TABLE_ID       = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_suppurch::LineItem-innerTable";
const PURCH_TITLE_ID       = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_suppurch::LineItem-title";
const PURCH_FILTERBAR_ID   = "zpmdc04displaysup::ZPMDC04_SUPPLIER_HEADER_CObjectPage--fe::table::_suppurch::LineItem-toolbar";

// onAfterRendering 내부
setTimeout(() => {
    ResponsiveTableRowCount.wireOnce(oView, COMPANY_TABLE_ID, COMPANY_TITLE_ID, COMPANY_FILTERBAR_ID);
    ResponsiveTableRowCount.wireOnce(oView, PURCH_TABLE_ID,   PURCH_TITLE_ID,   PURCH_FILTERBAR_ID);
}, 1000);
```

#### 주의사항

| # | 주의사항 |
|:---:|---|
| 1 | **테이블 ID 세그먼트는 Facet ID(대문자)가 아닌 `Navigation Path`(소문자, `_` 시작)를 사용한다.** |
| 2 | Object Page엔 FilterBar가 없으므로 4번째 파라미터에 **`-toolbar` ID**를 전달한다. |
| 3 | List Report와 달리 Object Page에는 **Variant 처리(`Variant.headerVariant/tableVariant`)를 적용하지 않는다.** |
| 4 | 자식 테이블이 여러 개면 `wireOnce`를 **테이블 개수만큼 반복 호출**한다. |
| 5 | GridTable/AnalyticalTable을 Object Page에서 쓸 경우도 동일 원칙(Navigation Path 사용, toolbar ID)을 적용한다. |

---

## 4. 파일 헤더 규칙

파일 생성 시 파일 유형에 따라 아래 두 가지 형식 중 하나를 사용합니다.

---

### 4-1. XML 파일 (Fragment / View) 헤더 형식

XML 파일 최상단 주석으로 추가합니다.

```xml
<!--
*******************************************************************
Module (Sub) : {모듈명}
Fragment    : {파일명}.fragment.xml
Description : {설명}
========================================
Modification LOG
001  {YYYY/MM/DD}  {사번}  Initial Release
*******************************************************************
-->
```

> ✅ 실제 적용 예시:
> ```xml
> <!--
> *******************************************************************
> Module (Sub) : FI
> Fragment    : CustomColumn.fragment.xml
> Description : UserDefFeild01 체크박스 표시용 커스텀 컬럼
> ========================================
> Modification LOG
> 001  2026/03/06  DFI005  Initial Release
> *******************************************************************
> -->
> ```

---

### 4-2. Controller (JS) 파일 헤더 형식

JS 파일 최상단 주석으로 추가합니다.

```javascript
/*
*******************************************************************
1. Module (Sub) : {모듈명}
2. App Id       : {AppId}
3. App Title    : {앱 제목}
4. File Name    : {파일명}.controller.js
5. OData        : {OData 서비스명} ({ODataVersion})
6. Page Type    : Fiori Elements (List Report | Object Page)
7. Description  : {설명}
*******************************************************************
Modification LOG
-------------------------------------------------------------------
No.   CSR/PJT No.        Date        Author     Description
----  ----------------  ----------  ---------  ------------------
001   {CSR번호}          {YYYY/MM/DD}  {사번}     Initial Release
*******************************************************************
Comment:

*******************************************************************
*/
```

> ✅ 실제 적용 예시:
> ```javascript
> /*
> *******************************************************************
> 1. Module (Sub) : CO
> 2. App Id       : zfcoc01mcstmt
> 3. App Title    : Manufacturing Cost Statement
> 4. File Name    : McstmtListExt01.controller.js
> 5. OData        : zfcoc01_mcstmt_srv (V4.0)
> 6. Page Type    : Fiori Elements (List Report Extension)
> 7. Description  : Manufacturing Cost Statement List Extension Controller
> *******************************************************************
> Modification LOG
> -------------------------------------------------------------------
> No.   CSR/PJT No.        Date        Author     Description
> ----  ----------------  ----------  ---------  ------------------
> 001   PP000017-25446    2026/05/06  DMM001     Initial Release
> *******************************************************************
> Comment:
>
> *******************************************************************
> */
> ```

---

## 5. Controller 처리 방식 (신규 vs 기존 파일)

---

### 5-1. 신규 생성 전체 템플릿 (파일이 없을 때)

> ⚠️ 파일이 없을 때는 아래 전체 템플릿을 그대로 사용한다. 플레이스홀더는 섹션 6의 규칙에 따라 실제 값으로 치환한다.

```javascript
/*
*******************************************************************
1. Module (Sub) : {모듈명}
2. App Id       : {AppId}
3. App Title    : {AppTitle}
4. File Name    : ListReportPageExt.controller.js
5. OData        : {OData서비스명} ({ODataVersion})
6. Page Type    : Fiori Elements (List Report Extension)
7. Description  : {AppTitle} List Extension Controller
*******************************************************************
Modification LOG
-------------------------------------------------------------------
No.   CSR/PJT No.        Date        Author     Description
----  ----------------  ----------  ---------  ------------------
001                     {YYYY/MM/DD}  {사번}     Initial Release
*******************************************************************
Comment:

*******************************************************************
*/
sap.ui.define([
    'sap/ui/core/mvc/ControllerExtension',
    "ext/lib/common/Variant",
    "ext/lib/common/{RowCountLib}"          // Case A/B/C에 따라 변경
], function (ControllerExtension, Variant, {RowCountLib}) {
    'use strict';

    const TBL_ID          = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable";
    const TITLE_ID          = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title";
    const FILTERBAR_ID      = "{AppId}::{EntityList}--fe::table::{Entity}::FilterBar";
    // GridTable 전용 (Case B에만 추가) — 반드시 `LineItem-filterInfoBar` 사용 (구버전 `::FilterInfoBar` 금지)
    // const FILTERBAR_INFO_ID = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-filterInfoBar";

    function downloadPDF() {
        const pdfUrl = "https://lginnotek.sharepoint.com/"; // 실제 메뉴얼 URL로 변경
        window.open(pdfUrl, "_blank");
    }

    return ControllerExtension.extend('{AppId}.ext.controller.ListReportPageExt', {
        override: {
            onInit: function () {
                var oModel = this.base.getExtensionAPI().getModel();
            },

            onAfterRendering: function () {
                const oView    = this.getView();
                const bManual  = false;
                const oTableVM = oView.byId("{AppId}::{EntityList}--fe::table::{Entity}::LineItem::VM");

                // ① VariantManagement 위치 이동
                Variant.headerVariant(this, bManual, downloadPDF);
                Variant.tableVariant(oTableVM);

                // ② Row Count 이벤트 연결
                setTimeout(() => {
                    {RowCountLib}.{wireMethod}(oView, TBL_ID, TITLE_ID, FILTERBAR_ID);
                }, 1000);
            }
        }
    });
});
```

> **`{RowCountLib}` / `{wireMethod}` 치환 규칙**
>
> | TableType | RowCountLib | wireMethod |
> |---|---|---|
> | ResponsiveTable (V4) | `ResponsiveTableRowCount` | `wireOnce` |
> | GridTable (V4) | `GridTableRowCount` | `wireOnce` |
> | AnalyticalTable (V2) | `AnalyticalTableRowCount` | `v2AnalyticalTablewireOnce` |

---

### 5-2. 기존 파일 수정 체크리스트 (파일이 이미 있을 때)

> ⚠️ 파일을 읽은 뒤 아래 체크리스트를 순서대로 확인하고, **없는 항목만 추가**한다. 기존 코드는 절대 삭제하지 않는다.

#### 체크 항목 및 처리 방법

| # | 확인 항목 | 없을 때 처리 방법 | 삽입 위치 |
|:---:|---|---|---|
| 1 | **파일 헤더 주석** (`/\* ****... \*/`) | 섹션 4-2 형식으로 추가 | 파일 **최상단** (1번째 줄) |
| 2 | **`"ext/lib/common/Variant"`** (sap.ui.define 모듈) | 모듈 배열 끝에 추가, 함수 인자에도 `Variant` 추가 | `sap.ui.define([...` 배열 |
| 3 | **`"ext/lib/common/{RowCountLib}"`** (sap.ui.define 모듈) | 모듈 배열 끝에 추가, 함수 인자에도 추가 | `sap.ui.define([...` 배열 |
| 4 | **상수 ID 블록** (`TBL_ID`, `TITLE_ID`, `FILTERBAR_ID` 등) | 섹션 3-3 Case에 맞게 추가 | `'use strict';` 바로 아래 |
| 5 | **`downloadPDF` 함수** | 상수 블록 바로 아래에 추가 | 상수 블록 아래, `return` 위 |
| 6 | **`onAfterRendering` 함수** | `override` 블록 내 `onInit` 뒤에 추가 | `onInit` 함수 닫는 `},` 바로 뒤 |
| 7 | **`onAfterRendering` 내 Variant 초기화** (이미 함수가 있는 경우) | 함수 내부 첫 줄에 Variant 코드 추가 | `onAfterRendering: function () {` 바로 아래 |
| 8 | **`onAfterRendering` 내 setTimeout RowCount** (이미 함수가 있는 경우) | 함수 내부 끝에 setTimeout 블록 추가 | Variant 코드 아래 |

#### 기존 파일 수정 예시 (before → after)

**Before (Fiori Elements 기본 생성 파일):**
```javascript
sap.ui.define(['sap/ui/core/mvc/ControllerExtension'], function (ControllerExtension) {
    'use strict';

    return ControllerExtension.extend('{AppId}.ext.controller.ListReportPageExt', {
        override: {
            onInit: function () {
                var oModel = this.base.getExtensionAPI().getModel();
            }
        }
    });
});
```

**After (수정 완료):**
```javascript
/*
*******************************************************************
1. Module (Sub) : {모듈명}
...헤더 추가...
*******************************************************************
*/
sap.ui.define([
    'sap/ui/core/mvc/ControllerExtension',
    "ext/lib/common/Variant",               // ← 추가
    "ext/lib/common/{RowCountLib}"          // ← 추가
], function (ControllerExtension, Variant, {RowCountLib}) {
    'use strict';

    // ← 상수 블록 추가
    const TABLE_ID     = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-innerTable";
    const TITLE_ID     = "{AppId}::{EntityList}--fe::table::{Entity}::LineItem-title";
    const FILTERBAR_ID = "{AppId}::{EntityList}--fe::table::{Entity}::FilterBar";

    // ← downloadPDF 추가
    function downloadPDF() {
        const pdfUrl = "https://lginnotek.sharepoint.com/";
        window.open(pdfUrl, "_blank");
    }

    return ControllerExtension.extend('{AppId}.ext.controller.ListReportPageExt', {
        override: {
            onInit: function () {
                var oModel = this.base.getExtensionAPI().getModel(); // ← 기존 코드 유지
            },

            // ← onAfterRendering 추가
            onAfterRendering: function () {
                const oView    = this.getView();
                const bManual  = false;
                const oTableVM = oView.byId("{AppId}::{EntityList}--fe::table::{Entity}::LineItem::VM");

                Variant.headerVariant(this, bManual, downloadPDF);
                Variant.tableVariant(oTableVM);

                setTimeout(() => {
                    {RowCountLib}.{wireMethod}(oView, TBL_ID, TITLE_ID, FILTERBAR_ID);
                }, 1000);
            }
        }
    });
});
```

---

## 6. manifest.json 커스텀 컬럼 Excel Export 패턴

> ⚠️ **이 섹션은 새 List Report Extension Controller 생성과는 무관하며, 커스텀 컬럼(fragment 기반)을 Excel Export에 포함시킬 때만 적용한다.**

### 배경 및 원인

SAP Fiori Elements에서 `template`(fragment) 기반 커스텀 컬럼은 **FE Export 엔진이 렌더링된 UI 컨텐츠를 읽지 못한다.**
아래 두 가지 속성을 **반드시 함께** 지정해야 Export에 포함된다.

| 속성 | 역할 | 없을 때 결과 |
|---|---|---|
| `properties` | FE 프레임워크에 이 컬럼이 어떤 OData 필드를 참조하는지 선언 | Export 엔진이 컬럼 자체를 무시 → **컬럼 누락** |
| `exportSettings` | 엑셀에 출력할 실제 필드명/레이블/타입 지정 | 컬럼은 포함되나 값이 비어서 출력될 수 있음 |

### 적용 패턴 (manifest.json)

```json
"columns": {
    "DataField::{fieldName}": {
        "availability": "Hidden"
    },
    "MyCustomColumn": {
        "header": "{컬럼헤더}",
        "template": "{AppId}.ext.fragment.{FragmentName}",
        "position": {
            "placement": "Before",
            "anchor": "DataField::{fieldName}"
        },
        "properties": [
            "{fieldName}"
        ],
        "availability": "Default",
        "width": "8rem",
        "exportSettings": {
            "property": "{fieldName}",
            "label": "{컬럼헤더}",
            "type": "String"
        }
    }
}
```

### 적용 규칙

1. `DataField::{fieldName}` 원본 컬럼은 `"availability": "Hidden"` 으로 화면에서 숨긴다.
2. 커스텀 컬럼에 **`properties: ["{fieldName}"]`** 반드시 추가한다. ← **Export 포함 여부 결정**
3. 커스텀 컬럼에 **`exportSettings`** 반드시 추가한다. ← **Export 출력값 지정**
4. `exportSettings.property` 값은 `properties` 배열의 필드명과 동일하게 지정한다.

### 실제 적용 예시 (Nation 커스텀 컬럼)

```json
"columns": {
    "DataField::nation": {
        "availability": "Hidden"
    },
    "NationFlagColumn": {
        "header": "Nation",
        "template": "zffic30verifycrettest6.ext.fragment.NationFlag",
        "position": {
            "placement": "Before",
            "anchor": "DataField::nation"
        },
        "properties": [
            "nation"
        ],
        "availability": "Default",
        "width": "8rem",
        "exportSettings": {
            "property": "nation",
            "label": "Nation",
            "type": "String"
        }
    }
}
```

---

## 7. 플레이스홀더 치환 규칙

> ✅ `{AppId}`, `{EntityList}`, `{Entity}`, `{OData 서비스명}`, `{TableType}` 값은
> **반드시 `webapp/manifest.json`을 읽어서 자동 추출**한 실제 값으로 치환합니다. (섹션 0 참조)

| 플레이스홀더 | 설명 | 추출 방법 | 현재 프로젝트 값 |
|---|---|---|---|
| `{AppId}` | 앱 ID | `manifest.json` → `sap.app.id` | `zpmdc01salbomcreate` |
| `{EntityList}` | List Report 타겟 ID | `manifest.json` → `routing.targets`의 List target `id` | `ZPMDC01_SALESBOMHEADER_CList` |
| `{Entity}` | 엔티티명 | `manifest.json` → List target `contextPath`에서 `/` 제거 | `ZPMDC01_SALESBOMHEADER_C` |
| `{OData 서비스명}` | OData 서비스명 | `manifest.json` → `dataSources.mainService.uri`에서 추출 | `zpmdc01_salbomcreate_ui_v4` |
| `{TableType}` | 테이블 유형 | `manifest.json` → `tableSettings.type` | `ResponsiveTable` |
| `{ODataVersion}` | OData 버전 | `manifest.json` → `dataSources.mainService.settings.odataVersion` | `4.0` (V4) |
| `{사번}` | 작성자 사번 | 고정값 | `DMM002` |

---

## 8. Excel Upload (ExcelTemplate 공통 라이브러리)

### 8-1. 개요

`ext/lib/common/ExcelTemplate` 공통 라이브러리를 사용하여 Excel 파일 업로드 및 미리보기 다이얼로그를 구현합니다.

- 사용자가 Excel 파일을 선택하면 OData Action을 호출하여 서버에서 데이터를 파싱
- 파싱된 결과를 미리보기 테이블에 표시한 후 저장 가능

### 8-2. 사전 준비 (공통 Library 환경설정)

**섹션 2 (공통 Lib 등록 처리)** 가 완료되어 있어야 합니다.
- `Component.js`에 `jQuery.sap.registerModulePath` 등록
- `manifest.json`의 `sap.ui5 > dependencies > libs`에 `"ext.lib.common": {}` 추가

### 8-3. 파일 위치

```
webapp/ext/controller/ExcelTemplate.js        ← 업로드 이벤트 컨트롤러
webapp/ext/fragment/UploadFile.fragment.xml   ← 업로드 다이얼로그 Fragment (필요 시)
webapp/Template/<FileName>.xlsx               ← 다운로드용 Excel 템플릿 파일
```

### 8-4. sap.ui.define 의존성 추가

```javascript
sap.ui.define([
    "sap/m/MessageToast",
    "sap/m/MessageBox",
    "sap/ui/core/Fragment",
    "ext/lib/common/ExcelTemplate"
], function(MessageToast, MessageBox, Fragment, ExcelTemplate) {
    'use strict';
```

### 8-5. onExcelUpload 구현

#### 필수 파라미터 목록

| 파라미터 | 타입 | 설명 |
|---|---|---|
| `this` | Object | 컨트롤러 컨텍스트 |
| `oView` | Object | `this.getView()` |
| `sTableTitle` | String | 미리보기 다이얼로그 테이블 제목 |
| `aCols` | Array | 미리보기 테이블 컬럼 정의 |
| `sEntity` | String | OData 엔티티 경로 (예: `/ZFCOC31_FundsCenterAuthority_C`) |
| `sActionName` | String | 업로드 OData Action 전체 이름 |
| `aActionParams` | Array | Action 파라미터 배열 `[{param, value}]` |
| `fnInvoke` | Function | 업로드 Action 완료 후 콜백 |
| `aButtons` | Array | 다이얼로그 버튼 정의 `[{text, type, pressEvent}]` |
| `iDialogWidth` | String | 다이얼로그 너비 (예: `"1700px"`) |

#### aCols 컬럼 타입 종류

| type 값 | 설명 |
|---|---|
| `"Text"` | 일반 텍스트 |
| `"Icon"` | 아이콘 (MESSAGE_TYPE 등) |
| `"ObjectStatus"` | 상태 표시 (status 속성 추가 필요) |

#### 전체 코드 예시

```javascript
onExcelUpload: async function() {
    const oView = this.getView();
    const sTableTitle = "Preview Data - Default View";
    const sEntity = "/<EntitySetName>";
    const sActionName = "<OData_Namespace>.<ActionName>(...)";
    const aActionParams = [
        { param: "contentS", value: "" }
    ];
    const iDialogWidth = "1700px";

    // 업로드 OData Action 완료 후 콜백
    const fnInvoke = function(fileEvent, TBL_ID, oAction, resetExcelList) {
        const oResult = oAction.getBoundContext().getObject();

        if (oResult) {
            const oTempModel = oView.getModel("uploadTemp");
            if (oTempModel) {
                const aUploadedData = oResult.value;
                oTempModel.setProperty("/ExcelTempData", aUploadedData);

                const oExcelUploadTable = oView.byId(TBL_ID);
                oExcelUploadTable.setModel(oTempModel);

                MessageToast.show("Please review the upload list.");
            } else {
                resetExcelList(oView, TBL_ID);
                MessageBox.error("File upload failed.");
            }
        } else {
            resetExcelList(oView, TBL_ID);
            MessageBox.error("File upload failed.");
        }
    };

    // 저장 버튼 클릭 시 실행 함수
    const fnSave = function() {
        const oModel = this.getModel();
        const oTempModel = oView.getModel("uploadTemp");
        const oFilterBar = oView.byId("FilterBar");
        const aExcelData = oTempModel.getProperty("/ExcelTempData");
        const sSaveActionName = "<OData_Namespace>.<SaveActionName>(...)";
        const oListBinding = oModel.bindList("/<EntitySetName>");
        const oCollectionContext = oListBinding.getHeaderContext();
        const oSaveAction = oModel.bindContext(sSaveActionName, oCollectionContext);

        oSaveAction.setParameter("_Item", aExcelData);

        oSaveAction.invoke()
            .then(function() {
                this._pUploadDlg.then(function(oDialog) {
                    const iCloseButtonIndex = oDialog.getButtons().length - 1;
                    oDialog.getButtons()[iCloseButtonIndex].firePress();
                    oFilterBar.triggerSearch();
                    MessageToast.show("Data saved successfully.");
                }.bind(this));
            }.bind(this))
            .catch(function(oError) {
                let sMessage = "Action failed";
                if (oError && oError.message) {
                    sMessage = oError.message;
                }
                MessageBox.error(sMessage);
                ExcelTemplate.clearFileUploader(oView);
            }.bind(this))
            .finally(function() {
                // TODO
            }.bind(this));
    };

    // 다이얼로그 버튼
    const aButtons = [
        { text: "저장", type: "Emphasized", pressEvent: fnSave }
    ];

    // 미리보기 테이블 컬럼 정의
    const aCols = [
        { label: "Result Message", property: "MESSAGE_TYPE", textAlign: "Center", width: "50px",  type: "Icon" },
        { label: "Row",            property: "Sequence",     textAlign: "Begin",  width: "50px",  type: "Text" },
        { label: "FM Area",        property: "FinancialManagementArea", textAlign: "End", width: "130px", type: "Text" },
        { label: "Employee Number",property: "EmployeeNumber", textAlign: "Begin", width: "130px", type: "Text" },
        { label: "Employee Name",  property: "EmployeeName",  textAlign: "Begin", width: "130px", type: "Text" },
        { label: "Funds Center",   property: "FundsCenter",   textAlign: "Begin", width: "150px", type: "Text" },
        { label: "Result",         property: "CheckResult",   textAlign: "Begin", width: "80px",  type: "ObjectStatus", status: "CheckResult" },
        { label: "Message",        property: "Message",       textAlign: "Begin", width: "200px", type: "Text" }
    ];

    ExcelTemplate.onExcelUpload(
        this, oView, sTableTitle, aCols,
        sEntity, sActionName, aActionParams,
        fnInvoke, aButtons, iDialogWidth
    );
},
```

### 8-6. onDownloadTemplate 구현

Excel 템플릿 파일을 다운로드하는 함수입니다.

```javascript
onDownloadTemplate: async function() {
    ExcelTemplate.onDownloadTemplate("<appId>/Template/<FileName>.xlsx");
},
```

- `<appId>`: `manifest.json`의 `sap.app > id` 값 (예: `zfcoc31fcauth`)
- 템플릿 파일은 `webapp/Template/` 폴더에 위치해야 합니다.

### 8-7. View에서 버튼 연결 예시

```xml
<MenuButton id="btnImport" text="Import" enabled="true" visible="{BtnMode>/isEditable}">
    <menu>
        <Menu>
            <MenuItem
                icon="sap-icon://upload"
                text="Import From Excel"
                press=".onImportExcel" />
            <MenuItem
                icon="sap-icon://download"
                text="Download Template"
                press=".onDownloadTemplate" />
        </Menu>
    </menu>
</MenuButton>
```

### 8-8. 주의사항

- `fnInvoke` 내부에서 `MessageBox`는 모듈로 주입된 `MessageBox`를 사용하세요. (`sap.m.MessageBox` 전역 참조 지양)
- `fnSave`는 일반 function으로 정의하여 `ExcelTemplate` 공통 라이브러리가 `this` 컨텍스트를 바인딩할 수 있도록 합니다.
- `uploadTemp` 모델은 공통 라이브러리에서 자동으로 생성·관리합니다.
- `oSaveAction.invoke()`는 OData V4 기준이며, 이전 버전에서는 `execute()`를 사용합니다.

---

## 9. LineItem Action Toolbar에 MenuButton 추가 (ControllerExtension)

### 9-1. 개요

Fiori Elements List Report의 LineItem Action Toolbar에 `MenuButton`을 동적으로 추가합니다.
`manifest.json`을 수정하지 않고 **ControllerExtension**의 `onAfterRendering`에서 프로그래밍 방식으로 추가합니다.

> ⚠️ **주의**: `ActionToolbar`은 `addContent()` 직접 호출이 **금지**되어 있습니다.
> 반드시 `sap/ui/mdc/actiontoolbar/ActionToolbarAction`으로 감싸서 `addAction()`을 사용해야 합니다.

### 파일 위치
```
webapp/ext/controller/<ProjectName>ListExt01.controller.js
```

### 9-2. sap.ui.define에 모듈 추가

```javascript
sap.ui.define([
    'sap/ui/core/mvc/ControllerExtension',
    // ... 기존 모듈 ...
    'sap/m/MenuButton',
    'sap/m/Menu',
    'sap/m/MenuItem',
    'sap/ui/mdc/actiontoolbar/ActionToolbarAction'
], function (ControllerExtension, /* ... */, MenuButton, Menu, MenuItem, ActionToolbarAction) {
```

### 9-3. onAfterRendering에 호출 추가

```javascript
onAfterRendering: function () {
    // ... 기존 코드 ...

    // LineItem Action Toolbar에 MenuButton 추가
    this._addExcelMenuButton(oView);
}
```

### 9-4. _addExcelMenuButton 구현

- `override` 블록 **외부**에 추가
- Toolbar ID 패턴: `fe::table::<EntityName>::LineItem-toolbar`
- 중복 방지: `sap.ui.getCore().byId(sActionId)` 체크

```javascript
_addExcelMenuButton: function (oView) {
    var oToolbar = oView.byId("fe::table::<EntityName>::LineItem-toolbar");
    if (!oToolbar) {
        return;
    }

    // 중복 방지
    var sActionId = oView.createId("excelMenuButtonAction");
    if (sap.ui.getCore().byId(sActionId)) {
        return;
    }

    var oThis = this;
    var oMenuButton = new MenuButton(oView.createId("excelMenuButton"), {
        text: "Import",
        defaultAction: function () {
            oThis.onExcelUpload();
        },
        menu: new Menu({
            items: [
                new MenuItem({
                    icon: "sap-icon://upload",
                    text: "Import From Excel",
                    press: function () {
                        oThis.onExcelUpload();
                    }
                }),
                new MenuItem({
                    icon: "sap-icon://download",
                    text: "Download Template",
                    press: function () {
                        oThis.onDownloadTemplate();
                    }
                })
            ]
        })
    });

    // ActionToolbar은 addContent() 직접 호출 금지 → ActionToolbarAction으로 감싸서 addAction() 사용
    var oAction = new ActionToolbarAction(sActionId, {
        action: oMenuButton
    });

    oToolbar.addAction(oAction);
},
```

### 9-5. MenuItem 구성

| 순서 | icon | text | press |
|---|---|---|---|
| 1st | `sap-icon://upload` | Import From Excel | `onExcelUpload` |
| 2nd | `sap-icon://download` | Download Template | `onDownloadTemplate` |

### 9-6. 주의사항

| 항목 | 내용 |
|---|---|
| `addContent()` 금지 | `ActionToolbar`은 Fiori Elements 내부 컨트롤로 직접 content 추가 불가 → 에러 발생 |
| `ActionToolbarAction` 필수 | `sap/ui/mdc/actiontoolbar/ActionToolbarAction`으로 감싸서 `addAction()` 사용 |
| 중복 방지 | `onAfterRendering`은 여러 번 호출될 수 있으므로 ID 체크로 중복 방지 |

### 9-7. 전체 코드 예시

```javascript
sap.ui.define([
    'sap/ui/core/mvc/ControllerExtension',
    'ext/lib/common/Variant',
    'ext/lib/common/ResponsiveTableRowCount',
    'sap/m/MessageToast',
    'sap/m/MessageBox',
    'sap/ui/core/Fragment',
    'ext/lib/common/ExcelTemplate',
    'sap/m/MenuButton',
    'sap/m/Menu',
    'sap/m/MenuItem',
    'sap/ui/mdc/actiontoolbar/ActionToolbarAction'
], function (ControllerExtension, Variant, ResponsiveTableRowCount, MessageToast, MessageBox, Fragment, ExcelTemplate, MenuButton, Menu, MenuItem, ActionToolbarAction) {
    'use strict';

    return ControllerExtension.extend('<appId>.ext.controller.<ProjectName>ListExt01', {
        override: {
            onAfterRendering: function () {
                Variant.headerVariant(this, false, this.downloadPDF);

                var oTblVM = this.base.getView().byId("fe::table::<EntityName>::LineItem::VM");
                Variant.tableVariant(oTblVM);

                var oView = this.base.getView();
                var TBL_INNER_ID      = oView.byId("fe::table::<EntityName>::LineItem-innerTable").getId();
                var TBL_TITLE_ID      = oView.byId("fe::table::<EntityName>::LineItem-title").getId();
                var TBL_FILTERINFO_ID = oView.byId("fe::table::<EntityName>::LineItem-filterInfoBar").getId();
                setTimeout(function () {
                    const oFilterBar = oView.findAggregatedObjects(true, o => o.isA && o.isA("sap.ui.mdc.FilterBar"))[0];
                    ResponsiveTableRowCount.wireOnce(oView, TBL_TITLE_ID, oFilterBar.getId());
                }, 2000);

                // LineItem Action Toolbar에 MenuButton 추가
                this._addExcelMenuButton(oView);
            }
        },

        _addExcelMenuButton: function (oView) {
            var oToolbar = oView.byId("fe::table::<EntityName>::LineItem-toolbar");
            if (!oToolbar) { return; }

            var sActionId = oView.createId("excelMenuButtonAction");
            if (sap.ui.getCore().byId(sActionId)) { return; }

            var oThis = this;
            var oMenuButton = new MenuButton(oView.createId("excelMenuButton"), {
                text: "Import",
                defaultAction: function () { oThis.onExcelUpload(); },
                menu: new Menu({
                    items: [
                        new MenuItem({
                            icon: "sap-icon://upload",
                            text: "Import From Excel",
                            press: function () { oThis.onExcelUpload(); }
                        }),
                        new MenuItem({
                            icon: "sap-icon://download",
                            text: "Download Template",
                            press: function () { oThis.onDownloadTemplate(); }
                        })
                    ]
                })
            });

            var oAction = new ActionToolbarAction(sActionId, { action: oMenuButton });
            oToolbar.addAction(oAction);
        },

        onExcelUpload: async function () { /* 섹션 8 참조 */ },
        onDownloadTemplate: async function () { /* 섹션 8 참조 */ }
    });
});
```
