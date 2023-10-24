현재 회사에서는 Vue 프레임워크를 사용하지만 vuetify는 사용하지 못한다. 그래서 기본적인 컴포넌트들도 직접 html과 css로 만들고 있다.
어느날 약 40문항이 있는 설문지 같은 화면을 개발해야했다. 경험상 label이 붙는 input은 어렵진 않지만 코드는 길어지고 계속 id와 for로 짝을 지어줘야해서 아주 귀찮으면서도 손 많이가는 작업이라고 생각한다.
그동안은 화면에서 1~2개 그룹 이상은 쓰이지 않아서 그냥 input-label로 계속 개발했는데 이참에 다른 개발자들의 편의를 생각해서라도 라디오그룹용 컴포넌트를 하나 만들어야 겠다고 생각했다.

## 라디오그룹 컴포넌트 만들기
내가 생각한 구조는 일단 ```Array```에 ```value```와 ```label```의 값이 들어있는 ```Object```형태로, 해당 라디오그룹의 ```select``` 값은 ```v-model```로 주고받는 구조였다.
``` html
// Main.vue
<base-radio-group
	:data-items="[{value : 'Dog', label: '강아지'}, 
    			  {value : 'Cat', label: '고양이'}, 
                  {value : 'Bear', label: '곰'}]"
    v-model="petAns">
</base-radio-group>
```

``` html
// BaseRadioGroup.vue
<template>
  <span v-for="(item, index) in dataItems" :key="item.value">
    <input type="radio"
           :id="`${randomId}_${index}`"
           :value="item.value"
           :checked="item.value === value"
           @change="$emit('m-change', $event.target.value)"
    >
    <label :for="`${randomId}_${index}`">{{ item.label }}</label>
  </span>
</template>

<script>
export default {
  name: 'BaseRadioGroup',
  model: {
    prop: 'value',
    event: 'm-change'
  },
  props: {
    value : { },
    dataItems: {
      type : Array,
      required : false,
      default: function (){
        return[
          {value: "Y", label: "Yes"},
          {value: "N", label: "No"},
        ]
      }
    }
  },
  created () {
    this.randomId = Math.random() * 1000000000000
  },
  data () {
    return {
      randomId : ''
    }
  },
}
</script>
```

```randomId```는 원래 생각지도 못했는데 dataItems의 index로 id를 세팅해주니 한 화면에서 ```<base-radio-group>``` 을 사용하니 id가 겹쳐버렸다. 그래서 random으로 id를 생성해서 컴포넌트별로 각각의 id를 가지도록 했다.

## 라디오그룹 컴포넌트 옵션 추가하기
쓰다보니 disabled 조건도 필요하고 다국어도 처리해야하고 dataItems Array의 구조를 계속 value와 label형태로 만들어주는게 너무 번거로웠다. 그래서 컴포넌트를 좀 더 편리하게 쓰기위해 다양한 props를 추가하여 사용성을 높였다.
``` html
// Main.vue
<base-radio-group
    :data-items="[ {code: '01', ko: '강아지', en: 'dog' },
                    {code: '02', ko: '고양이', en: 'cat'},
                    {code: '03', ko: '곰', en: 'bear'}]"
    :disabed="!hasPet"
    :value-field="'code'"
    :label-field="$t('pet.text')"
    v-model="petCode">
</base-radio-group>
```
``` html
// BaseRadioGroup.vue
<template>
  <span v-for="(item, index) in dataItems" :key="item.value">
    <input type="radio"
           :id="`${randomId}_${index}`"
           :disabled="disabled"
           :value="valueField ? item[valueField] : item.value"
           :checked="valueField ? item[valueField] : item.value) === value"
           @change="$emit('m-change', $event.target.value)"
    >
    <label :for="`${randomId}_${index}`">
      {{ labelField ? item[labelField] : item.label}}
    </label>
  </span>
</template>

<script>
export default {
  name: 'BaseRadioGroup',
  model: {
    prop: 'value',
    event: 'm-change'
  },
  props: {
    value : {},
    disabled : {
      type : Boolean,
      default : false
    },
    dataItems: {
      type : Array,
      default: function (){
        return[
          {value: "Y",label: "Yes"},
          {value: "N",label: "No"}
        ]
      }
    },
    labelField : {
      type : String,
      default : 'label'
    },
    valueField : {
      type : String,
      default : 'value'
    }
  },
  mounted () {
    this.randomId = Math.random() * 10000
  },
  data () {
    return {
      randomId : '',
    }
  },
}
</script>
```

이외에도 click event를 추가하는 등 더 다양한 기능을 만들었지만 이정도만 있어도 개발할 때 더 확실해 편해지리라고 확신한다. 설문지를 다 개발한 후에 dataItems는 그대론데 형태만 selectbox로 바꿔달라는 요구사항이 들어왔다. selectbox는 이미 위와 같은 구조의 컴포넌트가 있었기 때문에 props들은 다 그대로 두고 컴포넌트만 바꿔주기만 해도 되서 ```<base-radio-group>```을 만들기 아주 잘했다는 생각이 든다!

다음에는 input-label의 영원한 짝궁.. checkbox에 대한 컴포넌트도 만들어보고자한다.
