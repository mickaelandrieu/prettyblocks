<script setup>
import { ref, computed } from "vue"
import { useLeftPanelStore } from '../store/index'
import Icon from './Icon.vue'
import ButtonLight from './ButtonLight.vue';
import emitter from 'tiny-emitter/instance'
import { contextShop, useStore } from '../store/currentBlock'
import axios from 'axios'

const props = defineProps({
  id: String, // unique id for each block
  title: String,
  icon: String,
  config: Boolean,
  element: Object,
  is_child: Boolean
})

// used to get selected element in panel
const store = useStore()
const removeSubState = () => {
  let context = contextShop()
  const params = {
    formattedID: props.id,
    action: 'removeSubState',
    ajax: true,
    ctx_id_lang: context.id_lang,
    ctx_id_shop: context.id_shop,
    ajax_token: security_app.ajax_token
  }
  axios.get(ajax_urls.state, { params }).then((response) => response.data)
    .then((data) => {
      if (props.element.need_reload) {
        emitter.emit('reloadIframe', props.element.id_prettyblocks)
      }
      emitter.emit('stateUpdated', props.element.id_prettyblocks)
      emitter.emit('initStates')
      emitter.emit('displayState', props.element)
      emitter.emit('forceSave', props.element.id_prettyblocks)
    })
}

const removeState = async () => {
  if (confirm('Voulez vous supprimer l\'element ?')) {
    const params = {
      id_prettyblocks: props.element.id_prettyblocks,
      action: 'removeState',
      ajax_token: security_app.ajax_token
    }
    let response = await axios.get(ajax_urls.state, { params })
    emitter.emit('initStates')
    emitter.emit('reloadIframe', null)
  }
  // emitter.emit('reloadIframe', null)
}

// when we select a element in list
function select(instance) {
  // set store current block name
  store.$patch({
    id_prettyblocks: parseInt(props.id.split('-')[0]),
    subSelected: props.id
  })
  //  emitter.emit('displayState', props.element.id_prettyblocks)
}

const isSelected = computed(() => props.id == store.subSelected)

// when we click on the eye icon to disable element
const disabled = ref(false)
</script>

<template>
  <div
    :class="['menu-item flex items-center px-2 py-1 mb-1 rounded-md hover:bg-gray-100 border-2 border-transparent cursor-pointer', { 'selected': isSelected }]"
    @click="select">
    <!-- this slot is used to add extra action on the left, for example the collapse icon -->
    <slot></slot>
    <!-- icon and name of item -->
    <div :class="['flex items-center flex-grow pr-2', { disabled }]">
      <!-- {{ element }} -->
      <Icon :name="icon" class="h-5 w-5 mr-2"></Icon>
      <p class="flex-grow w-0 text-ellipsis whitespace-nowrap overflow-hidden select-none">
        {{ title }}
      </p>
    </div>
    <!-- extra actions : eye and drag buttons -->
    <div class="menu-item-actions w-0 overflow-hidden flex justify-end items-center">
      <!-- {{ props.is_child }} -->
      <ButtonLight class="handle" v-if="props.config" icon="CogIcon" />
      <ButtonLight class="handle" v-if="props.is_child" @click.prevent="removeSubState" icon="TrashIcon" />
      <ButtonLight class="handle" v-if="!props.is_child" @click.prevent="removeState" icon="TrashIcon" />
      <!-- <ButtonLight @click="disabled = !disabled" :icon="disabled ? 'EyeOffIcon' : 'EyeIcon'" /> -->
      <ButtonLight class="handle cursor-move" icon="ArrowsUpDownIcon" />
    </div>
  </div>
</template>

<style scoped>
.menu-item.selected {
  @apply bg-indigo bg-opacity-10 text-indigo
}

.menu-item .disabled {
  @apply transition duration-200;
  @apply opacity-70
}

.menu-item:hover>.menu-item-actions {
  @apply w-auto;
}
</style>
