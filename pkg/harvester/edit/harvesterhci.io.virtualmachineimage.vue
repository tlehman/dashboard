<script>
import CruResource from '@shell/components/CruResource';
import Tabbed from '@shell/components/Tabbed';
import Tab from '@shell/components/Tabbed/Tab';
import { LabeledInput } from '@components/Form/LabeledInput';
import KeyValue from '@shell/components/form/KeyValue';
import NameNsDescription from '@shell/components/form/NameNsDescription';
import { RadioGroup } from '@components/Form/Radio';
import LabelValue from '@shell/components/LabelValue';
import Select from '@shell/components/form/Select';
import LabeledSelect from '@shell/components/form/LabeledSelect';

import CreateEditView from '@shell/mixins/create-edit-view';
import { OS } from '../mixins/harvester-vm';
import { VM_IMAGE_FILE_FORMAT } from '../validators/vm-image';
import { HCI as HCI_ANNOTATIONS } from '@shell/config/labels-annotations';
import { exceptionToErrorsArray } from '@shell/utils/error';
import { allHash } from '@shell/utils/promise';
import { STORAGE_CLASS } from '@shell/config/types';
import { HCI } from '../types';

const DOWNLOAD = 'download';
const UPLOAD = 'upload';
const rawORqcow2 = 'raw_qcow2';

export default {
  name: 'EditImage',

  components: {
    Tab,
    Tabbed,
    KeyValue,
    Select,
    CruResource,
    LabeledInput,
    NameNsDescription,
    RadioGroup,
    LabelValue,
    LabeledSelect,
  },

  mixins: [CreateEditView],

  props: {
    value: {
      type:     Object,
      required: true,
    },
  },

  async fetch() {
    const inStore = this.$store.getters['currentProduct'].inStore;

    await allHash({
      images:         this.$store.dispatch(`${ inStore }/findAll`, { type: HCI.IMAGE }),
      storageClasses: this.$store.dispatch(`${ inStore }/findAll`, { type: STORAGE_CLASS }),
    });

    const defaultStorage = this.$store.getters[`${ inStore }/all`](STORAGE_CLASS).find(s => s.isDefault);

    this.$set(this, 'storageClassName', this.storageClassName || defaultStorage?.metadata?.name || 'longhorn');
  },

  data() {
    if ( !this.value.spec ) {
      this.$set(this.value, 'spec', { sourceType: DOWNLOAD });
    }

    if (!this.value.metadata.name) {
      this.value.metadata.generateName = 'image-';
    }

    return {
      url:         this.value.spec.url,
      files:       [],
      resource:    '',
      headers:     {},
      fileUrl:     '',
      file:        '',
    };
  },

  computed: {
    uploadFileName() {
      return this.file?.name || '';
    },

    imageName() {
      return this.value?.metadata?.annotations?.[HCI_ANNOTATIONS.IMAGE_NAME] || '-';
    },

    isCreateEdit() {
      return this.isCreate || this.isEdit;
    },

    showEditAsYaml() {
      return this.value.spec.sourceType === DOWNLOAD;
    },

    storageClassOptions() {
      const inStore = this.$store.getters['currentProduct'].inStore;
      const storages = this.$store.getters[`${ inStore }/all`](STORAGE_CLASS);

      const out = storages.filter(s => !s.parameters?.backingImage).map((s) => {
        const label = s.isDefault ? `${ s.name } (${ this.t('generic.default') })` : s.name;

        return {
          label,
          value: s.name,
        };
      }) || [];

      return out;
    },

    storageClassName: {
      get() {
        return this.value.metadata.annotations[HCI_ANNOTATIONS.STORAGE_CLASS];
      },

      set(nue) {
        this.value.metadata.annotations[HCI_ANNOTATIONS.STORAGE_CLASS] = nue;
      }
    },
  },

  watch: {
    'value.spec.url'(neu) {
      const url = neu.trim();

      this.setImageLabels(url);
    },

    'value.spec.sourceType'() {
      this.$set(this, 'file', null);
      this.url = '';

      if (this.$refs?.file?.value) {
        this.$refs.file.value = null;
      }
    },
  },

  methods: {
    async saveImage(buttonCb) {
      this.value.spec.displayName = (this.value.spec.displayName || '').trim();

      if (this.value.spec.sourceType === UPLOAD && this.isCreate) {
        try {
          this.value.spec.url = '';

          const file = this.file;

          this.value.metadata.annotations[HCI_ANNOTATIONS.IMAGE_NAME] = file?.name;

          const res = await this.value.save();

          res.uploadImage(file);

          buttonCb(true);
          this.done();
        } catch (e) {
          this.errors = exceptionToErrorsArray(e);
          buttonCb(false);
        }
      } else {
        this.save(buttonCb);
      }
    },

    setImageLabels(str) {
      const suffixName = str?.split('/')?.pop() || str;

      const fileSuffix = suffixName?.split('.')?.pop()?.toLowerCase();

      if (VM_IMAGE_FILE_FORMAT.includes(fileSuffix)) {
        const labelValue = fileSuffix === 'iso' ? fileSuffix : rawORqcow2;

        this.addLabel(HCI_ANNOTATIONS.IMAGE_SUFFIX, labelValue);

        if (!this.value.spec.displayName) {
          this.$refs.nd.changeNameAndNamespace({
            text:     suffixName,
            selected: this.value.metadata.namespace,
          });
        }
      }

      const os = this.getOSType(str);

      if (os) {
        this.addLabel(HCI_ANNOTATIONS.OS_TYPE, os.value);
      }
    },

    addLabel(labelKey, value) {
      const rows = this.$refs.labels.rows;

      rows.map((label, idx) => {
        if (label.key === labelKey) {
          this.$refs.labels.remove(idx);
        }
      });
      this.$refs.labels.add(labelKey, value);
    },

    handleFileUpload() {
      const file = this.$refs.file.files[0];

      this.file = file;

      this.setImageLabels(file?.name);

      if (!this.value.spec.displayName) {
        this.$refs.nd.changeNameAndNamespace({
          text:     file?.name,
          selected: this.value.metadata.namespace,
        });
      }

      this.setImageLabels();
    },

    selectFile() {
      // Clear the value so the user can reselect the same file again
      this.$refs.file.value = null;
      this.$refs.file.click();
    },

    internalAnnotations(option) {
      const optionKeys = [HCI_ANNOTATIONS.OS_TYPE, HCI_ANNOTATIONS.IMAGE_SUFFIX];

      return optionKeys.find(O => O === option.key);
    },

    calculateOptions(keyName) {
      if (keyName === HCI_ANNOTATIONS.OS_TYPE) {
        return OS;
      } else if (keyName === HCI_ANNOTATIONS.IMAGE_SUFFIX) {
        return [{
          label: 'ISO',
          value: 'iso'
        }, {
          label: 'raw/qcow2',
          value: rawORqcow2
        }];
      }

      return [];
    },

    focusKey() {
      this.$refs.key.focus();
    },

    getOSType(str) {
      if (!str) {
        return;
      }

      return OS.find( (os) => {
        if (os.match) {
          return os.match.find(matchValue => str.toLowerCase().includes(matchValue)) ? os.value : false;
        } else {
          return str.toLowerCase().includes(os.value.toLowerCase()) ? os.value : false;
        }
      });
    }
  },
};
</script>

<template>
  <CruResource
    :done-route="doneRoute"
    :resource="value"
    :mode="mode"
    :errors="errors"
    :can-yaml="showEditAsYaml ? true : false"
    :apply-hooks="applyHooks"
    @finish="saveImage"
  >
    <NameNsDescription
      ref="nd"
      v-model="value"
      :mode="mode"
      :label="t('generic.name')"
      name-key="spec.displayName"
    />

    <Tabbed v-bind="$attrs" class="mt-15" :side-tabs="true">
      <Tab
        name="basic"
        :label="t('harvester.image.tabs.basics')"
        :weight="99"
        class="bordered-table"
      >
        <RadioGroup
          v-if="isCreate"
          v-model="value.spec.sourceType"
          name="model"
          :options="[
            'download',
            'upload',
          ]"
          :labels="[
            t('harvester.image.sourceType.download'),
            t('harvester.image.sourceType.upload'),
          ]"
          :mode="mode"
        />
        <div class="row mb-20 mt-20">
          <div v-if="isCreateEdit" class="col span-12">
            <LabeledInput
              v-if="isEdit"
              v-model="value.spec.sourceType"
              :mode="mode"
              class="mb-20"
              :disabled="isEdit"
              label-key="harvester.image.source"
            />

            <LabeledInput
              v-if="value.spec.sourceType === 'download'"
              v-model="value.spec.url"
              :mode="mode"
              :disabled="isEdit"
              class="labeled-input--tooltip"
              required
              label-key="harvester.image.url"
              :tooltip="t('harvester.image.urlTip', {}, true)"
            />

            <div v-else>
              <button
                v-if="isCreate"
                type="button"
                class="btn role-primary"
                @click="selectFile"
              >
                <span>
                  {{ t('harvester.image.uploadFile') }}
                </span>
                <input
                  v-show="false"
                  id="file"
                  ref="file"
                  type="file"
                  accept=".qcow, .qcow2, .raw, .img, .iso"
                  @change="handleFileUpload()"
                />
              </button>

              <div
                v-if="uploadFileName"
                class="fileName mt-5"
              >
                <span class="icon icon-file" />
                {{ uploadFileName }}
              </div>
            </div>
          </div>
          <div
            v-else
            class="col span-12"
          >
            <LabelValue
              :name="t('harvester.image.fileName')"
              :value="imageName"
            />
          </div>
        </div>
      </Tab>

      <Tab
        name="storage"
        :label="t('harvester.storage.label')"
        :weight="89"
        class="bordered-table"
      >
        <div class="row">
          <div class="col span-6">
            <LabeledSelect
              v-model="storageClassName"
              :options="storageClassOptions"
              :label="t('harvester.storage.storageClass.label')"
              :mode="mode"
              :disabled="isEdit"
              class="mb-20"
            />
          </div>
        </div>
      </Tab>

      <Tab name="labels" :label="t('labels.labels.title')" :weight="2" class="bordered-table">
        <KeyValue
          key="labels"
          ref="labels"
          :value="value.labels"
          :add-label="t('labels.addLabel')"
          :mode="mode"
          :pad-left="false"
          :read-allowed="false"
          @focusKey="focusKey"
          @input="value.setLabels($event)"
        >
          <template #key="{ row, keyName, queueUpdate}">
            <input
              ref="key"
              v-model="row[keyName]"
              :placeholder="t('keyValue.keyPlaceholder')"
              @input="queueUpdate"
            />
          </template>

          <template #value="{row, keyName, valueName, queueUpdate}">
            <Select
              v-if="internalAnnotations(row)"
              v-model="row[valueName]"
              :searchable="true"
              :clearable="false"
              :options="calculateOptions(row[keyName])"
              @input="queueUpdate"
            />
            <input
              v-else
              v-model="row[valueName]"
              :disabled="isView"
              :type="'text'"
              :placeholder="t('keyValue.valuePlaceholder')"
              autocorrect="off"
              autocapitalize="off"
              spellcheck="false"
              @input="queueUpdate"
            />
          </template>
        </KeyValue>
      </Tab>
    </Tabbed>
  </CruResource>
</template>
