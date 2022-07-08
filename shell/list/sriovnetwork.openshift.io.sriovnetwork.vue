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

export default {
  name:       'HarvesterListSriovNetworks',
  components: {},

  fetch() {
    this.sriovNetworks = [];
  },

  data() {
    return {
      hash:                  {},
      rows:                  [],
      hosts:                 [],
      sriovNetworks:          [],
    };
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
