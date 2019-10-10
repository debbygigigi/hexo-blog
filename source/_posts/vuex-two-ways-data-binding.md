---
title: vuex two ways data binding
date: 2018-12-19 23:05:59
tags:
- Vue
- Vuex
- DataBinding
---

我們都知道vue很大的特性就是可以做到 [two-way data binding](https://vuejs.org/v2/guide/forms.html) ，並且很簡單，只要透過 `v-model` 綁定 data的值即可，如下

```htmlmixed=
<input v-model="message" placeholder="edit me">
```

```javascript=
new Vue({
  data: {
    message: ''
  }
})
```

除了form元件（input, textarea, and select）之外，[custom component](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components) 也可以做到。

不過 Vuex 也想做到 `two-way data binding` 該怎麼做呢

## 原來的做法

假設目前有一個 component 和一個 vuex ，component 會需要從 vuex 取值也會賦值，也就是想要實現 `two-way data binding`

簡單的 vuex 範例，state 裡存放 filters 的狀態，透過 actions 跟 mutations 去呼叫並改變 state 值，如果會 vuex 的話應該看得懂

*Store.js*
```javascript=
const state = {
    filters: {
        type: 'rent',
        regions: ['七堵區'],
        keyword: '123'
    },
};

const actions = {
    setFilter({state, commit}, filter) {
        commit('SET_FILTER', filter);
    }
};

const mutations = {
    SET_FILTER: (state, filters) => {
        _.forEach(filters, (value, key) => {
            state.filters[key] = value;
        });
    }
};
```

這邊是精簡後的 component


*vue component*

```javascript=
<template>
    <select v-model="regions" />
</template>

export default {
    data() {
        return {
            regions: []
        };
    },
    computed: {
        ...mapState({
            regionsState: state => state.filters.regions
        }),
    },
    watch: {
        regions(value) {
            this.setFilter({'regions': value});
        }
    },
    created() {
        this.regions = this.regionsState;
    },
    methods: {
        ...mapActions(['setFilter']),
    }
};
```

來解釋一下，在這個component中，引入了另一個 custom component: select，並跟 data 綁了 v-model
由於要實現雙向綁定，分別從vuex取值及賦值

取值：先從 `mapState` 取值，並在 `created` 階段時將值給了 data，於是select component便得到初始值
賦值：利用 `watch` 監聽 data 值，只要值有變動，就執行 `mapActions` 去更動vuex的state

## 調整

經過調整後，程式碼如下

```javascript=
<template>
    <select v-model="regions" />
</template>

export default {
    // data() {
    //     return {
    //         regions: []
    //     };
    // },
    computed: {
        ...mapState({
            regionsState: state => state.Search.filters.regions
        }),
        regions: {
            get() {
                return this.regionsState;
            },
            set(value) {
                this.setFilter({'regions': value});
            }
        }
    },
    // watch: {
    //     regions(value) {
    //         this.setFilter({'regions': value});
    //     }
    // },
    // created() {
    //     this.regions = this.regionsState;
    // },
    methods: {
        ...mapActions('Search', ['setFilter']),
    }
}
```

一樣是 v-model綁值，只是綁的值改在computed，利用 computed 的 set與get特性來做取值與賦值

取值：從 computed 的 `get` 取得 `mapState`
賦值：computed 的值若有變動，在 `set` 去執行 `mapActions`

這樣就可以少了 data, watch, created 這些程式碼，利用 computed 去做這些事

## 結尾

在會用 vuex 後，其實都是用第一種作法，或是剛好都沒用到 `v-model`
最近在用的時候靈機一想並上網搜尋後，才發現有這樣好用的做法，並且官方 [Vuex - Form Handling](https://vuex.vuejs.org/guide/forms.html) 其實也有提到

看來真的沒事就要看看官方文件 :smile:

## 參考

* [Form Fields, Two-Way Data Binding and Vuex](https://markus.oberlehner.net/blog/form-fields-two-way-data-binding-and-vuex/)
* [Anyway, this is how to use v-model with Vuex. Computed setter in action.](https://itnext.io/anyway-this-is-how-to-use-v-model-with-vuex-computed-setter-in-action-320eb682c976)
* [Vuex - Form Handling](https://vuex.vuejs.org/guide/forms.html)