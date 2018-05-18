<template>
  <div class="tree-view-wrapper">
    <input v-if="this.enableFilter" placeholder="Search element" v-model="filter">
    <button v-if="this.collapse" type="button" class="btn" @click="collapseAll">Collapse All</button>
    <tree-view-item class="tree-view-item-root" :data="parsedData" :max-depth="allOptions.maxDepth" :current-depth="0" :modifiable="allOptions.modifiable" @change-data="onChangeData"></tree-view-item>
  </div>
</template>

<script>
  import _ from 'lodash'
  import TreeViewItem from './TreeViewItem.vue'

  export default {
    components:{
      TreeViewItem
    },
  	name: "tree-view",
    props: ["data", "options", "collapse", "enableFilter"],
    data(){
      return {
        filter: ''
      }
    },
    methods: {

    	// Transformer for the non-Collection types,
      // like String, Integer of Float
      transformValue: function(valueToTransform, keyForValue, filter_string){
      	return {
        	key: keyForValue,
          type: "value",
          value: valueToTransform,
          filter: filter_string
        }
      },

    	// Since we use lodash, the _.map method will work on
      // both Objects and Arrays, returning either the Key as
      // a string or the Index as an integer
    	generateChildrenFromCollection: function(collection, filter_string){
  			return _.map(collection, (value, keyOrIndex)=>{
            if (this.isObject(value)) {
              return this.transformObject(value, keyOrIndex, false, filter_string);
            }
            if (this.isArray(value)) {
              return this.transformArray(value, keyOrIndex, filter_string);
            }
            if (this.isValue(value)) {
              return this.transformValue(value, keyOrIndex, filter_string);
            }
          }) ;
      },

    	// Transformer for the Array type
      transformArray: function(arrayToTransform, keyForArray, filter_string){
      	return {
        	key: keyForArray,
          type: "array",
          children: this.generateChildrenFromCollection(arrayToTransform, filter_string)
        }
      },

      // Transformer for the Object type
    	transformObject: function(objectToTransform, keyForObject, isRootObject = false, filter_string=''){
        return {
        	key: keyForObject,
        	type: "object",
          isRoot: isRootObject,
          children: this.generateChildrenFromCollection(objectToTransform, filter_string),
          filter: filter_string
        }
      },

      // Helper Methods for value type detection
      isObject: function(value){
      	return _.isPlainObject(value);
      },

      isArray: function(value){
      	return _.isArray(value);
      },

      isValue: function(value){
      	return !this.isObject(value) && !this.isArray(value);
      },

      onChangeData: function(path, value) {
        let lastKey = _.last(path)
        path = _.dropRight(_.drop(path))

        let data = _.cloneDeep(this.data)
        let targetObject = data
        _.forEach(path, (key) => {
          targetObject = targetObject[key]
        })

        if (targetObject[lastKey] != value) {
          targetObject[lastKey] = value
          this.$emit('change-data', data)
        }
      },
      collapseAll: function(){
        _.map(this.$children, child => {
          child.open = false;
          child.collapseAll();
        });
      }
    },
    computed: {
      allOptions: function(){
        return _.extend({}, {
          rootObjectKey:  "root",
          maxDepth:       4,
          modifiable:     false
        }, (this.options || {}) )
      },
    	parsedData: function(){
      	// Take the JSON data and transform
        // it into the Tree View DSL

        // Strings or Integers should not be attempted to be split, so we generate
        // a new object with the string/number as the value
        if (this.isValue(this.data)) {
          return this.transformValue(this.data, this.allOptions.rootObjectKey, this.filter);
        }

        // If it's an object or an array, transform as an object
  	    return this.transformObject(this.data, this.allOptions.rootObjectKey, true, this.filter);
      }
    }
  };
</script>

<style scoped>

.tree-view-wrapper {
  overflow: auto;
}

/* Find the first nested node and override the indentation */
.tree-view-item-root > .tree-view-item-leaf > .tree-view-item {
  margin-left: 0!important;
}

/* Root node should not be indented */
.tree-view-item-root {
  margin-left: 0!important;
}

</style>
