<script>
import { mapGetters } from 'vuex';
import AsyncButton from '@shell/components/AsyncButton';
import { Card } from '@components/Card';
import { Banner } from '@components/Banner';
import { exceptionToErrorsArray } from '@shell/utils/error';
export default {
  components: {
    Card,
    AsyncButton,
    Banner,
  },
  props:      {
    vm: {
      type:     Object,
      required: true
    },
  },
  data() {
    return { errors: [], resolve: null };
  },
  computed: { ...mapGetters({ t: 'i18n/t' }) },
  methods:  {
    close() {
      this.resolve();
      this.$emit('close');
    },
    apply(buttonDone) {
      try {
        this.vm.doActionGrowl('restart', {});
        buttonDone(true);
        this.resolve();
        this.close();
      } catch (err) {
        console.error(err); // eslint-disable-line
        this.errors = exceptionToErrorsArray(err);
        buttonDone(false);
      }
    }
  }
};
</script>

<template>
  <modal
    class="restart-modal"
    name="restartDialog"
    :width="600"
    height="auto"
    :click-to-close="false"
    @closed="close"
  >
    <Card class="prompt-restore" :show-highlight-border="false">
      <h4 slot="title" class="text-default-text" v-html="t('harvester.modal.restart.title')" />

      <template slot="body">
        <slot name="body">
          <div class="pl-10 pr-10" v-html="t('harvester.modal.restart.tip')">
          </div>
        </slot>
      </template>

      <div slot="actions" class="bottom">
        <Banner v-for="(err, i) in errors" :key="i" color="error" :label="err" />
        <div class="buttons">
          <button class="btn role-secondary mr-10" @click="close">
            {{ t('harvester.modal.restart.cancel') }}
          </button>

          <AsyncButton
            mode="restart"
            @click="apply"
          />
        </div>
      </div>
    </Card>
  </modal>
</template>
<style lang='scss' scoped>
  .prompt-restore {
    margin: 0;
  }
  .bottom {
    display: flex;
    flex-direction: column;
    flex: 1;
    .banner {
      margin-top: 0
    }
    .buttons {
      display: flex;
      justify-content: flex-end;
      width: 100%;
    }
  }
</style>
