<template>
  <div class="relog">
    <div class="d-md-none">
      <div class="menu-button" v-b-modal.modal-menu>Меню</div>
      <b-modal id="modal-menu" hide-footer>
        <List/>
      </b-modal>
    </div>
    <div class="col-12 d-none d-md-block col-md-3">
      <List/>
    </div>
    <div class="col-12 col-md-9 px-0">
      <Map/>
    </div>
  </div>
</template>

<script>
import Map from '@/components/Map'
import List from '@/components/List'

export default {
  name: 'Main',
  components: {Map, List},
  props: {},
  data() {
    return {}
  },
  mounted() {
  },
  methods: {}
}

</script>

<style lang="scss" scoped>
.menu-button {
  position: absolute;
  top: 0;
  left: 0;
  background: white;
  z-index: 999;
  width: fit-content;
}
</style>
