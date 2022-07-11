<script>

import { NAME, NAMESPACE } from '@shell/config/table-headers';
import { HCI, SCHEMA } from '@shell/config/types';

const schema = {
  id:         HCI.SRIOV_NETWORK,
  type:       SCHEMA,
  attributes: {
    kind:       HCI.SRIOV_NETWORK,
    namespaced: true
  },
  metadata: { name: HCI.SRIOV_NETWORK }
};

const sleep = (milliseconds) => {
  return new Promise(resolve => setTimeout(resolve, milliseconds));
};

export default {
  name:       'HarvesterListSriovNetworks',
  components: {},

  async fetch() {
    await sleep(1);
    this.sriovNetworks = [];
  },

  data() {
    return {};
  },

  computed: {
    headers() {
      return [
        NAME,
        NAMESPACE,
      ];
    },

    schema() {
      return schema;
    },

  },

  typeDisplay() {
    return this.$store.getters['type-map/labelFor'](schema, 99);
  },
};
</script>

<template>
  <div>List of SR-IOV Networks</div>
</template>
