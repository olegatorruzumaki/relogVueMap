<template>
  <DynamicScroller
      :items="parsedJson"
      :min-item-size="32"
      class="list-block"
  >
    <template v-slot="{ item, index, active }">
      <DynamicScrollerItem
          class="p-2 cursor-pointer border"
          :item="item"
          :active="active"
          :size-dependencies="[item.name]"
          :data-index="index"
      >
        <div v-b-toggle="`collapse-${index}`" class="text">{{ item.name }}</div>
        <b-collapse :id="`collapse-${index}`" class="ps-2" accordion="accordion" role="tabpanel">
          <DynamicScroller
              :items="item.apps"
              :min-item-size="54"
              class="scroller"
          >
            <template v-slot="{ item, index, active }">
              <DynamicScrollerItem
                  :item="item"
                  :active="active"
                  :size-dependencies="[item.type]"
                  :data-index="index"
              >
                <div @click="flyTo(item.coords)" class="border my-1 p-2 cursor-pointer">
                  <div>Тип: {{ item.type }}</div>
                  <div>Цена: {{ item.price }}</div>
                </div>
              </DynamicScrollerItem>
            </template>
          </DynamicScroller>
        </b-collapse>
      </DynamicScrollerItem>
    </template>
  </DynamicScroller>
</template>

<script>
import jsonApps from '../assets/json/apps.json'
import jsonClients from '../assets/json/clients.json'
import {eventBus} from '@/main'

export default {
  name: 'Main',
  components: {},
  props: {},
  data() {
    return {
      jsonApps: jsonApps,
      jsonClients: jsonClients,
      parsedJson: [],
    }
  },
  mounted() {
    let groupResult = jsonApps.reduce((obj, item) => {
      obj[item.client_id] = obj[item.client_id] || {};
      (obj[item.client_id][item.id] = obj[item.client_id][item.id] || []).push({
        id: item.id,
        type: item.type,
        client_id: item.client_id,
        price: item.price,
        coords: item.coords
      });
      return obj;
    }, {}); // Группирую apps по id client

    for (let client of jsonClients) { // Создаю новый массив клиентов с включенными apps
      let clientItem = client;
      clientItem.apps = Object.values(groupResult[client.id]).flat();

      this.parsedJson.push(clientItem);
    }

  },
  methods: {
    flyTo(data) {
      this.$bvModal.hide('modal-menu')
      eventBus.$emit('flyTo', data)
    }
  }
}

</script>

<style lang="scss" scoped>
.list-block {
  height: 95vh;
  overflow-y: scroll;
}
</style>
