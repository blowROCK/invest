<template>
  <!-- <div class="Products">아이템</div> -->
  <div id="invest">
    <div class="invest-header">
      <div class="invest-type">
        <div class="typeTitle"><h2 class="bold">상품유형</h2></div>
        <label class="allBox pointer">
          <input
            type="checkbox"
            class="pointer"
            v-model="allContract"
            @change="selectAllContract"
          />
          <h3>전체</h3>
          </label>
        <div class="bar"></div>
        <div class="checkboxes">
          <label class="pointer" v-for="(contract, index) in contractTypes" :key="index">
            <input
              type="checkbox"
              class="pointer"
              v-model="filters.contractType"
              :value="contract"
            />
            <h3>{{ contract }}</h3>
          </label>
        </div>
      </div>

      <div class="invest-type">
        <div class="typeTitle"><h2 class="bold">채권상태</h2></div>
        <label class="allBox pointer"
          ><input
            type="checkbox"
            class="pointer"
            v-model="allStatus"
            @change="selectAllStatus"
          />
          <h3>전체</h3>
          </label>
        <div class="bar"></div>
        <div class="checkboxes">
          <label class="pointer" v-for="(status, index) in statusTypes" :key="index">
            <input
              type="checkbox"
              class="pointer"
              v-model="filters.typedStatus"
              :value="status"
            />
            <h3>{{ status }}</h3>
          </label>
        </div>
      </div>
      <div class="invest-selected">
        <div
          class="selectedItem"
          v-for="(selected, index) in updateFilters"
          :key="index"
        >
          <h2>{{ selected }}</h2>
          <div class="times" @click="unCheck(selected)"></div>
        </div>
      </div>
    </div>

    <div class="invest-contents">
      <div class="invest-result">
        <div class="result">
          <h2>
            총 
              <span class="number">{{ listLength }}</span>
            건의 상품이 검색되었습니다.
          </h2>
        </div>
        <div class="orders">
          <h2 class="order pointer" @click="toggleOrder(false)" :class="{active : !orderByRate}">기본순</h2>
          <div class="bar"></div>
          <h2 class="order pointer" @click="toggleOrder(true)" :class="{active : orderByRate}">모집률</h2>
        </div>
      </div>
      <div class="invest-list">
        <article
          class="invest-card pointer"
          v-for="invest in investFilteredList"
          :key="invest.id"
        >
          <div class="thumnail">
            <div class="badge" v-if="invest.currentRate > 90"><p>마감임박</p></div>
            <img :src="invest.url" />
          </div>
          <div class="wrap-progress">
            <div class="bubble" v-bubble="invest.currentRate">
              {{ invest.currentRate }}% 달성
            </div>
            <div class="bar" :style="{ width: invest.currentRate + '%' }"></div>
          </div>
          <div class="contents">
            <div class="wrap-tags">
              <div class="tag">{{ invest.contractType }}</div>
              <div class="tag">{{ invest.typedStatus }}</div>
            </div>
            <div class="wrap-data">
              <div class="main-data">{{ invest.rateOfReturn }}%</div>
              <div class="bar-data"></div>
              <div>{{ invest.loanPeriod }}개월</div>
              <div class="bar-data"></div>
              <div>{{ invest.grade }}</div>
              <div class="bar-data"></div>
              <div>{{ invest.targetAmount | currency }}</div>
            </div>
            <div class="wrap-info">
              <h1>{{ invest.title }}</h1>
            </div>
            <div class="wrap-buttons">
              <button class="pointer">상세보기</button>
              <button class="blue pointer">바로투자</button>
            </div>
          </div>
        </article>
      </div>
      <div class="more-button pointer" @click="addLimit(limit+5)">
        더보기 ({{this.limit/5}}/{{Math.ceil(listLength/5)}})
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Invest',
  props: {
    investList: Object,
  },
  data: function() {
    return {
      orderByRate: false,
      limit : 5,
      currentPage: 0,

      allContract: false,
      contractTypes: ['건축자금', '부동산담보'],

      allStatus: false,
      statusTypes: ['대기중', '모집중'],

      filters: {
        contractType: [],
        typedStatus: [],
      },
      listLength: 0
    };
  },
  created: function() {},
  computed: {
    investFilteredList: function() {
      let list = this.investList.list;
      let filters = this.filters;
      
      const filterKeys = Object.keys(filters);
      let filteredList = list.filter(function(investObj) {
        return filterKeys.every((key)=>{
          if(filters[key].length == 0) return true;
          return filters[key].includes(investObj[key]);
        })
      });
      if(this.orderByRate){
        filteredList = filteredList.sort(function(a,b){
          return b.currentRate - a.currentRate;
        })
      }
      this.updateLength(filteredList.length);

      return filteredList.slice(0, this.limit);
    },
    updateFilters: function() {
      this.resetList();
      let tmpFilter = [];
      var filters = this.filters;

      this.contractTypes.forEach((itemName) => {
        if (filters.contractType.find((element) => element == itemName)) {
          tmpFilter.push(itemName);
        }
      });

      this.statusTypes.forEach((itemName) => {
        if (filters.typedStatus.find((element) => element == itemName)) {
          tmpFilter.push(itemName);
        }
      });

      this.selecContract(filters.contractType.length == this.contractTypes.length);
      this.selecStatus(filters.typedStatus.length == this.statusTypes.length);
      return tmpFilter;
    }
  },
  methods: {
    updateLength : function(length){
      this.listLength = length;
    },
    unCheck: function(name){
      let filters = this.filters;
      for (const key in filters) {
        var index = filters[key].indexOf(name);
        if (index !== -1) filters[key].splice(index, 1);
      }
    },
    toggleOrder: function(bool){
      this.orderByRate = bool;
      this.resetList();
    },
    addLimit:function(num){
      if(num >= this.listLength+5) return;
      this.limit = num;
    },
    resetList: function(){
      this.addLimit(5);
    },
    selectAllContract: function() {
      this.filters.contractType = [];
      if (this.allContract) {
        this.filters.contractType.push(...this.contractTypes);
      }
    },
    selectAllStatus: function() {
      this.filters.typedStatus = [];
      if (this.allStatus) {
        this.filters.typedStatus.push(...this.statusTypes);
      }
    },

    selecContract: function(bool) {
      this.allContract = bool;
    },
    selecStatus: function(bool) {
      this.allStatus = bool;
    }

  },
  filters: {
    currency(number) {
      const num = Number(number);
      var unitWords = ['원', '만', '억', '조'];
      var splitUnit = 10000;
      var splitCount = unitWords.length;
      var resultArray = [];
      var resultString = '';

      for (let i = 0; i < splitCount; i++) {
        var unitResult =
          (num % Math.pow(splitUnit, i + 1)) / Math.pow(splitUnit, i);
        unitResult = Math.floor(unitResult);
        if (unitResult > 0) {
          resultArray[i] = unitResult;
        }
      }
      for (let i = 0; i < resultArray.length; i++) {
        if (!resultArray[i]) continue;
        resultString = String(resultArray[i]) + unitWords[i];
      }
      return resultString;
    },
  }
};

import Vue from 'vue';
Vue.directive('bubble', {
  bind: function(el, binding, vnode) {
    let percent = binding.value;

    percent != 0 ? el.classList.add('on') : '';

    if (percent < 11) {
      el.classList.add('left');
      el.style.left = percent + '%';
    } else if (percent > 90) {
      el.classList.add('right');
      el.style.left = 'calc(' + percent + '% - 62px)';
    } else {
      el.classList.add('center');
      el.style.left = 'calc(' + percent + '% - 31px)';
    }
    vnode.context[binding.arg] = binding.value;
  },
});
</script>

<style scoped>
@import url('Invest.css');
</style>
