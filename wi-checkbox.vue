<template>
  <div>
  <b-form-group>
  <template v-for="(ctgry, i) in ctgries" >
      <div :key="`ctgry-${ i + 1 }`" class="ctgry-item">
        <div
          :indeterminate="ctgry.indeterminate"
          @click="onChangeCtgry(i)"
        >
          <b-icon  v-if="!ctgry.checked" icon="chevron-right"></b-icon>
          <b-icon v-else icon="chevron-down" ></b-icon>
          {{ ctgry.name }}
        </div>
      </div>
    <transition name="fade"  :key="`subctgry-${ i }`">
      <b-form-group
        v-if="ctgry.subcategories"
        v-show="ctgry.checked"
        :key="`subctgry-${ i }`"
        class="ctgry-item__subctgry"
      >
      <div
        v-for="(subctgry, j) in ctgry.subcategories"
        :key="`subctgry-${ j + 1 }`"
      >
        <b-form-radio
          v-model="subctgriesActiveValue"
          name="ctgry-radio"
          :key="`subctgry-${ j + 1 }`"
          :value="`ctgry${i}-item${j}-$${subctgry.name}`"
          @change="setIndeterminate(i, j)"
        >
          {{ subctgry.name }}
        </b-form-radio>
      </div>
      </b-form-group>
    </transition>
  </template>
  </b-form-group>
  <a
    :href="'#' + $route.fullPath + '#reset-ctgries'"
    class="wi-link"
    v-show="subctgriesActive"
    @click.prevent="reset"
  >Сбросить</a>
  </div>
</template>

<script>
  import { BFormGroup } from 'bootstrap-vue';
  export default {
    name: 'wi-category-checkbox',
    components: { BFormGroup },
    data: () => ({
      query: { type: 'topics', method: 'get_categories' }, // запрос категорий
      ctgries: [], // список с категориями
      subctgriesActiveValue: '', //
      subctgriesActive: '', //Текущая активная категория для отправки родителю ($emit)
      categoryNameActive: '', //Текущее активное имя для отправки родителю ($emit)
      activeIndexCtgry: null, //Текущий индекс активной категории
      activeIndexSubctgry: null //Текущий индекс активной подкатегории
    }),
    created() {
      this.getCtgries();
    },
    methods: {
      initKey() {  //Для инициализации новых ключей this.ctgries таких как (checked, selected, indeterminate)
        let initSubcategoriesKey = (subcategories) => { //Для инициализации новых ключей подкатегорий
          subcategories.forEach(subcategory => {
            this.$set(subcategory, 'checked', false);
          })
        };
        let initCtgriesKey = (ctgry) => { //Для инициализации новых ключей
          this.$set(ctgry, 'checked', false);
          this.$set(ctgry, 'indeterminate', false);
          this.$set(ctgry, 'selected', []);
        };

        this.ctgries.forEach(ctgry => {
          initCtgriesKey(ctgry);
          if (ctgry.subcategories) 
            initSubcategoriesKey(ctgry.subcategories)
        });
      }, 
      async getCtgries() {
        const ctgries = await window._wi_client.getList( this.query );
        if (ctgries.status === 200) {
          this.ctgries = ctgries.data.categories;
          this.initKey();
        } 
        else this.ctgries = [];
      },
      onChangeCtgry(i) {
        if (this.ctgries[i].checked)
          this.ctgries[i].checked = false
        else
          this.ctgries[i].checked = true
      },
      setCurrentHref(i, j) { //Установка текущего имени активной категории/подкатегории/subctgriesActiveValue
        this.subctgriesActive = this.ctgries[i].subcategories[j].name;
        this.categoryNameActive = this.ctgries[i].name;
        this.subctgriesActiveValue = `ctgry${i}-item${j}-$${this.ctgries[i].subcategories[j].name}`
      },
      setCurrentIndex(i, j) { //Установка текущего индекса активной категории/подкатегории
        this.activeIndexCtgry     = i;
        this.activeIndexSubctgry  = j;
      },
      setIndeterminate(i, j) {
        this.setCurrentHref(i, j);
        
        if ( isNumber(this.activeIndexCtgry) && isNumber(this.activeIndexSubctgry)) {
          this.ctgries[this.activeIndexCtgry].subcategories[this.activeIndexSubctgry].checked = false;
        }
        this.setCurrentIndex(i, j);
        
        const total         = this.ctgries[i].subcategories.length,
              lenSelected   = this.ctgries[i].selected.length;

        if (lenSelected && lenSelected < total)
          this.ctgries[i].indeterminate = true;
        else
          this.ctgries[i].indeterminate = false;

        this.ctgries[i].subcategories[j].checked = true;
        this.$emit('click', this.emitClick());

        function isNumber(val) {
          return typeof val === "number"
        }
      },
      clearAll() {
        this.categoryNameActive = '';
        this.subctgriesActive = '';
        this.subctgriesActiveValue = '';
      },
      reset() {
          this.clearAll();
          this.ctgries.forEach(ctgry => {
            if (ctgry.checked) {
              ctgry.checked = false;
              ctgry.selected = [];
            }
          });
          this.ctgries[this.activeIndexCtgry].subcategories[this.activeIndexSubctgry].checked = false;
      },
      emitClick() {
        return {
          category: this.categoryNameActive,
          subcategory: this.subctgriesActive
        };
      }
    },
    watch: {
      ctgries: {
        handler() {
          this.$emit('click', this.emitClick());
        },
        deep: true
      },
    },
    
  };
</script>

<style>
  a.wi-link {
    color: rgb(45, 101, 140);
    border-bottom: 1px dashed rgb(45, 101, 140);
  }

  a.wi-link:hover {
    text-decoration: none;
    border-bottom-style: solid;
  }

  .ctgry-item {
    padding: 5px 0px;
    cursor: pointer;
  }

  .ctgry-icon{
    padding-right: 20px;
  }

  .ctgry-item__subctgry .custom-radio {
    margin: 7px 5px 7px 15px;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .3s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */ {
    opacity: 0;
  }
</style>
