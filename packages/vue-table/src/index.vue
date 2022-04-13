<script lang="ts" setup>
import {
  AnyGenerics,
  CreateTableFactoryOptions,
  Options,
  PartialKeys,
  Table,
  createTableInstance,
  init,
  TableFeature,
} from '@tanstack/table-core'
import { defineProps, computed, reactive } from 'vue'
export * from '@tanstack/table-core'
export { createCoreTable, createTableFactory }

type GenericItem {}
interface GenericProps<GenericItem> = {
  table: Table<GenericItem>,
  options: PartialKeys<
    Omit<
      Options<TGenerics>,
      keyof CreateTableFactoryOptions<any, any, any, any>
    >,
    'state' | 'onStateChange'
  >
}

const props = defineProps<GenericProps>();
const resolvedOptions: Options<GenericItem> = {
  ...(table.__options ?? {}),
  state: {} // dummy state,
  onStateChange: () => {}, // noop
  ...options,
}

// Create a new table instance and store it in state
const [instance] = computed(() => {
  return createTableInstance<GenericItem>(resolvedOptions)
})

const state = reactive(() => instance.initialState)

// Compose the default state above with any user state. This will allow the user
// to only control a subset of the state if desired.
instance.setOptions(prev => ({
  ...prev,
  ...options,
  state: {
    ...state,
    ...options.state,
  },
   // Similarly, we'll maintain both our internal state and any user-provided
    // state.
    onStateChange: updater => {
      state.value = updater
      options.onStateChange?.(updater)
    },
}))

return instance
</script>
