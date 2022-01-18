<template>
  <div 
    :style="panziStyle"
  >
    {{panzi.n}}
  </div>
</template>

<script>
export default {
  props:{
    panzi:{
      type: Object
    }
  },
  computed:{
    panziStyle(){
      const { width, height, top, left , color } = this.panzi
      return {
        width,
        height,
        top,
        left,
        background: color,
        position: 'fixed',
        borderRadius: '5px',
        textAlign: 'center'
      }
    }
  }
}
</script>

<style scoped>
  
</style>
