<script>
export default {
  components: {},

  props: {
    value: {
      type:     Object,
      required: true,
    }
  },

  data() {
    const config = JSON.parse(this.value?.spec?.config || '{}');

    return {
      config,
      type: 'SriovNetwork',
    };
  },

  created() {
    if (this.registerBeforeHook) {
      this.registerBeforeHook(this.updateBeforeSave);
    }
  },

  methods: {

    saveNetwork(buttonCb) {},

    input(neu) {},

    updateBeforeSave() {}
  }
};
</script>

<template>
  <div>Editing SR-IOV Networks</div>
</template>
