<script setup>
import { defineProps, ref, defineComponent } from 'vue'
import Icon from './Icon.vue'
import { currentZone } from '../store/currentBlock'
import Loader from './Loader.vue'
import axios from 'axios'
import emitter from 'tiny-emitter/instance'
import { createToaster } from "@meforma/vue-toaster";
import { contextShop } from '../store/currentBlock'

const toaster = createToaster({
  position: 'top'
});

defineProps({
  name: String,
  description: String,
  code: {
    type: String,
    required: true,
  },
  icon: {
    type: String,
    default: 'PuzzleIcon'
  },
})

defineComponent({
  Icon,
  Loader
})

let showLoader = ref(false)

const AddOnZOne = async (code) => {
  let current_zone = currentZone()
  showLoader.value = true
  let context = contextShop()
  const params = {
    action: 'insertBlock',
    code: code,
    zone_name: current_zone.name,
    ctx_id_lang: context.id_lang,
    ctx_id_shop: context.id_shop,
    ajax_token: security_app.ajax_token
  }
  let url = ajax_urls.block_action_urls
  let res = await axios.get(url, { params })
  let data = await res.data
  emitter.emit('toggleModal', current_zone.name)
  emitter.emit('initStates')
  toaster.show('Block inséré avec succès', {
    position: "top"
  })

  emitter.emit('reloadIframe', data.id_prettyblocks);
}
</script>

<template>
  <div class="flex items-center gap-x-2 p-4 bg-gray-100 hover:bg-gray-200 rounded cursor-pointer transition-colors" @click="AddOnZOne(code)">
    <Icon :name="icon" class="h-10 w-10 shrink-0 text-indigo" />
    <div class="flex-1">
      <h3 class="text-lg font-bold">{{ name }}</h3>
      <p class="text-sm text-gray-600">{{ description }}</p>
    </div>
  </div>
</template>
