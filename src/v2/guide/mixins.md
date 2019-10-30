---
title: Mixins
type: guide
order: 301
---

## Basics

Mixins бол дахин ашиглагдах функционалуудыг Vue components-аас салгах маш уян хатан арга юм. Mixin объект нь ямар ч компонентын option-уудыг агуулах боломжтой. Компонент mixin хэрэглэхэд mixin доторх бүх option-ууд нь компонентын option-уудтай холилдох болно.

<div class="vue-mastery"><a href="https://www.vuemastery.com/courses/next-level-vue/mixins" target="_blank" rel="noopener" title="Mixins Tutorial">Vue Mastery дээрх тайлбар бичлэгийг үзэх</a></div>

Жишээ:

``` js
// define a mixin object
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// define a component that uses this mixin
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```

## Option нэгтгэх

Mixin болон компонент давхардсан option-ууд агуулж байвал тэдгээр нь зохистой стратеги ашиглаж нэгтгэгддэг.

Жишээ нь, дата объектууд нь энэхүү нэгтгэх явцыг давахад компонентын дата нь давхарлалт үүсэх нөхцөлд давуу эрхтэй байна.

``` js
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```

Ижил нэртэй hook функцууд нь array-руу нэгтгэгддэг учир тэдгээр нь бүгд дуудагдах юм. Mixin-ы hook-үүд компонентын hook-үүдийн **өмнө** дуудагдах болно.

``` js
var mixin = {
  created: function () {
    console.log('mixin hook called')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('component hook called')
  }
})

// => "mixin hook called"
// => "component hook called"
```

Объект утгууд ашигладаг option-ууд, жишээ нь `methods`, `components`, `directives`, нь ижил объект дотор нэгтгэгдэнэ. Тэдгээр объектуудад түлхүүр үгийн давхардал үүсэх үед компонентын option-ууд давуу эрхийг авах болно:

``` js
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})

vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
```

`Vue.extend()` дээр үүнтэй ижил нэгтгэх стратеги ашиглагддагийг анхаараарай.

## Глобал Mixin

Та mixin-г глобал байдлаар ашиглах мөн боломжтой. Болгоомжтой ашиглаарай! Нэгэнт глобал mixin ашиглавал энэ нь түүнээс хойш үүсэх бүх Vue Instance-д нөлөөлнө. Зохистой хэрэглэвэл үүнийг өөрийн тусгай option-ууд процесс логикийг оруулахад хэрэглэх боломжтой.

``` js
// `myOption` тусгай option-д handler оруулах
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

<p class="tip">Vue instance-н ширхэг бүрд нөлөөлдөг, гадаад сан байсан ч, учир глобал mixin-г бага хэмжээгээр анхааралтай ашиглах хэрэгтэй. Ихэнх тохиолдолд дээр дүрсэлсэн жишээ шиг тусгай option-г зохицуулах зорилгод л ашиглах нь зүйтэй. Мөн үүнийг [Plugins](plugins.html) байдлаар хүргэх нь аппликешн хувилагдахаас сэргийлэх сайхан санаа юм.</p>

## Тустай Option Нэгтгэх Стратеги

Тусгай option-ууд нэгтгэгдэхдээ үндсэн стратеги болох өмнө байсныг дарж ажилладаг. Хэрвээ тусгай option-г тусгай логикоор нэгтгэхийг хүсвэл `Vue.config.optionMergeStrategies` дотор функцээ оруулж өгөх хэрэгтэй:

``` js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // return mergedVal
}
```

Ихэнх объект дээр суурилсан option-уудад `methods` дээр хэрэглэгддэгтэй ижил стратеги хэрэглэж болно:

``` js
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

Илүү ярвигтай жишээг [Vuex](https://github.com/vuejs/vuex)-н 1.x нэгтгэх стратегиас харах боломжтой:

``` js
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```
