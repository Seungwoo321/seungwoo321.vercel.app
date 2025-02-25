---
title: vue-pivottable v0.4.1 업데이트
date: '2022-01-23'
tags: ["vue-pivottable", "Vue"]
categories: Vue
permalink: /blog/:year/:month/:day/:title/
---

작년 오픈소스 아카데미 컨트리뷰톤 참여를 계기로 2019년 8월 22일 첫 커밋(Commit)으로 시작해 NPM에 배포해서 관리 중인
오픈소스 `vue-pivottable`의 가이드 문서를 README.md에서 VuePress 기반의 웹 사이트로 구성하고 있습니다.

문서를 업데이트하면서 이슈 해결과 코드 리팩토링도 함께 진행 중인데 이번에는 슬롯(slot)과 스콥-슬롯(scoped-slot) 사용과 TableRenderer.js를 활용하는 예제를 추가했습니다.

<!--more-->

## 슬롯(slot)과 스콥-슬롯(scoped-slot) 그리고 v-slot

슬롯(slot)은 컴포넌트를 렌더링(rendering) 할 때 부모 컴포넌트에서 HTML 템플릿 코드를 정의할 수 있습니다.
범위가 있는 슬롯 스콥-슬롯(scoped-slot)은 HTML 템플릿 코드뿐만 아니라 데이터도 부모 컴포넌트의 접근이 필요할 때 사용할 수 있습니다.
그런데 뷰(Vue) 2.6.0에서 이 두 가지 속성을 대체하는 새 디렉티브 `v-slot`이 추가되었습니다. 아직 삭제된 건 아니지만 슬롯과 스콥-슬롯 속성은 사라질 예정이라고 합니다.

### 사용한 코드 살펴보기

`vue-pivottable`에서는 여러가지 슬롯과 스콥-슬롯을 제공 하고 있습니다. 스콥-슬롯은
`<vue-pivottable></vue-pivottable>`에서는 제공하지 않고 `<vue-pivottable-ui></vue-pivottable-ui>`에서만 사용 가능합니다.

여기서 `output` 이름을 가진 스콥-슬롯은 데이터를 피벗 형태로 표현하기 위해서 사용하는 `PivotData`생성자의 인스턴스를 제공하도록 했고 이 스콥-슬롯은
피벗 테이블을 바로 렌더(render) 하지 않고 다른 행동을 할 것인지를 선택하는 UI를 제공할 수 있습니다.

예를 들어 너무 많은 데이터는 브라우저가 화면을 렌더 하지 못하고 멈추게 되므로 사전에 엑셀이나 CSV 형태로 내려주는 선택지를 보여줄 수 있습니다.

### 문제 파악

피벗 테이블을 렌더 할지 아니면 가공된 데이터를 부모 컴포넌트에게 넘겨줄지를 결정하는 주요 코드를 살펴보았습니다.

```js
const limitOver = outputScopedSlot && this.colLimit > 0 && this.rowLimit > 0 && (pivotData.getColKeys().length > this.colLimit || pivotData.getRowKeys().length > this.rowLimit)
```

행과 열의 숫자 제한을 의미하는 props인 `colLimit`과  `rowLimit`의 값이 0보다 크고 실제 피벗을 그리는 데이터에서 행과 열의 수를 확인하고 이중 하나라도 props에서 지정한 제한 값보다 크다면 `true` 값을 갖게 될 것입니다.

렌더링 하는 부분은 다음과 같습니다.

```js
  h('tr',
    [
      rowAttrsCell,
      outputSlot ? h('td', { staticClass: 'pvtOutput' }, outputSlot) : limitOver ? h('td', { staticClass: 'pvtOutput' }, outputScopedSlot({ pivotData: new PivotData(props) })) : outputCell
    ]
  )
```

> 한 줄로 작성한 코드가 항상 좋은 코드는 아닙니다.

```js
if (outputSlot) {
    return h('td', { staticClass: 'pvtOutput' }, outputSlot)
} else {
    if (limitOver) {
        return h('td', { staticClass: 'pvtOutput' }, outputScopedSlot({ pivotData: new PivotData(props) }))
    } else {
        return outputCell
    }
}

```

슬롯 `output`이 있다면 우선적으로 렌더하고 그렇지 않으면 `limitOver`가 `true`일 경우 스콥-슬롯 `output`을 통해서 부모 컴포넌트에 데이터를 넘겨줍니다. `limitOver`가 `false`이면 원래 제공하는 피벗 테이블 화면을 렌더 합니다.

가이드 문서를 작성하기 위해서 이렇게 다시 살펴보니 `output` 스콥-슬롯을 사용하려면 별도의 props인 `colLimit`와 `rowLimit`를 알고 있어야 하고 내부의 `limitOver`의 로직도 이해하고 있어야 하는 점이 아주 불합리하게 느껴졌습니다.

### 문제 해결 및 코드 리팩토링

`output` 스콥-슬롯으로 데이터를 받을지는 부모 컴포넌트에서 결정할 문제이므로 props의 `colLimit`과 rowLimit` 그리고 이와 관련된 불필요한 로직을 제거했습니다. 그리고 한 줄이라서 읽고 이해하기 힘들었던 렌더링 부분을 정리하면서 슬롯과 스콥-슬롯이 통합된 v-slot 디렉티브도 정상 동작하도록 개선했습니다.

```js
  h('tr',
    [
      rowAttrsCell,
      outputSlot ? h('td', { staticClass: 'pvtOutput' }, outputSlot) : undefined,
      outputScopedSlot && !outputSlot ? h('td', { staticClass: 'pvtOutput' }, outputScopedSlot({ pivotData })) : undefined,
      !outputSlot && !outputScopedSlot ? outputCell : undefined
    ]
  )
```

## 테이블 렌더러 사용 예제

Pivottable.js에서는 렌더러(renderer)별 UI 컴포넌트를  몇 가지를 제공하는데 가장 기본이 테이블과 히트맵 등 테이블 기반의 UI를 렌더 하는 로직들이 모여 있는 것이 TableRenderer.js입니다.

이 프로젝트에서는 Vue의 render function을 사용해서 뷰 컴포넌트 생성하는 함수들을 제공하는 형태로 작성을 했었습니다.

```js
export default {
  Table: makeRenderer({ name: 'vue-table' }),
  'Table Heatmap': makeRenderer({ heatmapMode: 'full', name: 'vue-table-heatmap' }),
  'Table Col Heatmap': makeRenderer({ heatmapMode: 'col', name: 'vue-table-col-heatmap' }),
  'Table Row Heatmap': makeRenderer({ heatmapMode: 'row', name: 'vue-table-col-heatmap' }),
  'Export Table TSV': TSVExportRenderer
}
```

위의 `output` 스콥-슬롯에서 다시 본래의 테이블을 보여주고 싶을 수 있습니다. 이때 TableRenderer.js의 렌더 함수를 사용할 수 있습니다.

```html:VuePivotableOutputEx.vue {11-35,43-44,48-49} linenumber
<template>
  <div>
    <vue-pivottable-ui
      :data="[
        { color: 'blue', shape: 'circle' },
        { color: 'red', shape: 'triangle' },
      ]"
      :rows="['color']"
      :cols="['shape']"
    >
      <template v-if="!loaded" v-slot:output="{ pivotData }">
        <div v-if="!viewTable">
          <div class="btn-group">
            <a class="btn btn-sm btn-primary" @click="showTable">
              View Table
            </a>
            <a class="btn btn-sm btn-warning" @click="otherAction(pivotData)"
              >Other action
            </a>
          </div>
        </div>
        <template v-else>
          <table-renderer
            v-if="pivotData.props.rendererName === 'Table'"
            :data="pivotData.props.data"
            :props="pivotData.props"
          >
          </table-renderer>
          <heatmap-renderer
            v-if="pivotData.props.rendererName === 'Table Heatmap'"
            :data="pivotData.props.data"
            :props="pivotData.props"
          >
          </heatmap-renderer>
        </template>
      </template>
    </vue-pivottable-ui>
    <button
      class="btn btn-sm btn-secondary mt-1"
      :disabled="!loaded"
      @click="reset"
    >
      <i class="fas fa-redo mr-25"></i>
      redo
    </button>
  </div>
</template>

<script>
import { VuePivottableUi, Renderer } from "vue-pivottable";
import "vue-pivottable/dist/vue-pivottable.css";
const HeatmapRenderer = Renderer.TableRenderer["Table Heatmap"];
const TableRenderer = Renderer.TableRenderer["Table"];
export default {
  components: {
    VuePivottableUi,
    HeatmapRenderer,
    TableRenderer,
  },
  data() {
    return {
      viewTable: false,
      loaded: false,
    };
  },
  methods: {
    showTable() {
      this.viewTable = !this.viewTable;
      this.loaded = true;
    },
    otherAction(pivotData) {
      alert(`All Total Count: ${pivotData.allTotal.count}`);
    },
    reset() {
      this.viewTable = false;
      this.loaded = false;
    },
  },
};
</script>
```

위 코드의 실행 결과는 [여기](https://codesandbox.io/s/vue-pivottable-ui-outputscopedslot-rcp9k?from-embed=&file=/src/App.vue)에서 확인할 수 있습니다.

## 마무리

이 글의 시작하기 전에는 0.3.98 버전이었으나 거듭 수정을 하면서 0.4.1버전까지 오게 되었습니다.
그동안 미루었던 문제들을 문서에 수록할 예제 코드를 작성하면서 해결했습니다.
남은 문서 작업이 마무리되고 아직 해결되지 않은 이슈들도 모두 처리할 때까지 지속적으로 기록을 남길 예정입니다.

---

## Reference

* [Vue.js RFC - 새로운 slot 문법](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md)
