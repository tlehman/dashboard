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
    return {};
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
