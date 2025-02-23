<script>
import { mapGetters } from 'vuex';
import { exceptionToErrorsArray } from '@shell/utils/error';
import { Card } from '@components/Card';
import { Banner } from '@components/Banner';
import AsyncButton from '@shell/components/AsyncButton';
import { LabeledInput } from '@components/Form/LabeledInput';
import { Checkbox } from '@components/Form/Checkbox';

export default {
  name:       'HarvesterPvcCloneDialog',
  components: {
    AsyncButton, Banner, Card, LabeledInput, Checkbox
  },
  props:      {
    resources: {
      type:     Array,
      required: true
    }
  },
  data() {
    return {
      name:      '',
      cloneData: true,
      errors:       []
    };
  },
  computed: {
    ...mapGetters({ t: 'i18n/t' }),
    actionResource() {
      return this.resources[0];
    },
    disableSave() {
      return this.cloneData && !this.name;
    }
  },
  methods: {
    close() {
      this.name = '';
      this.$emit('close');
    },
    async save(buttonCb) {
      if (!this.cloneData) {
        this.resources[0].goToClone();
        buttonCb(false);
        this.close();

        return;
      }

      try {
        const res = await this.actionResource.doAction('clone', { name: this.name });

        if (res._status === 200 || res._status === 204) {
          this.$store.dispatch('growl/success', {
            title:   this.t('harvester.notification.title.succeed'),
            message: this.t('harvester.modal.volumeClone.message.success', { name: this.name })
          }, { root: true });
          this.close();
          buttonCb(true);
        } else {
          const error = [res?.data] || exceptionToErrorsArray(res);

          this.$set(this, 'errors', error);
          buttonCb(false);
        }
      } catch (err) {
        const error = err?.data || err;
        const message = exceptionToErrorsArray(error);

        this.$set(this, 'errors', message);
        buttonCb(false);
      }
    }
  },
};
</script>
<template>
  <Card :show-highlight-border="false">
    <template #title>
      {{ t('harvester.modal.volumeClone.title') }}
    </template>
    <template #body>
      <Checkbox v-model="cloneData" class="mb-10" label-key="harvester.modal.cloneVM.type" />

      <LabeledInput
        v-show="cloneData"
        v-model="name"
        class="mb-20"
        :label="t('harvester.modal.volumeClone.name')"
        required
      />
    </template>
    <div slot="actions" class="actions">
      <div class="buttons">
        <button class="btn role-secondary mr-10" @click="close">
          {{ t('generic.cancel') }}
        </button>
        <AsyncButton
          mode="create"
          :disabled="disableSave"
          @click="save"
        />
      </div>
      <Banner v-for="(err, i) in errors" :key="i" color="error" :label="err" />
    </div>
  </Card>
</template>
<style lang="scss" scoped>
.actions {
  width: 100%;
}
.buttons {
  display: flex;
  justify-content: flex-end;
  width: 100%;
}
</style>
