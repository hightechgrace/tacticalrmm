<template>
  <q-dialog ref="dialog" @hide="onHide">
    <q-card class="q-dialog-plugin" style="width: 60vw">
      <q-bar>
        {{ title }}
        <q-space />
        <q-btn dense flat icon="close" v-close-popup>
          <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
        </q-btn>
      </q-bar>
      <q-form @submit="submit">
        <!-- model select -->
        <q-card-section>
          <q-select
            label="Target"
            :options="modelOptions"
            map-options
            emit-value
            outlined
            dense
            :disable="editing"
            v-model="localField.model"
            :rules="[val => !!val || '*Required']"
          />
        </q-card-section>
        <!-- name -->
        <q-card-section>
          <q-input label="Name" outlined dense v-model="localField.name" :rules="[val => !!val || '*Required']" />
        </q-card-section>
        <!-- type select -->
        <q-card-section>
          <q-select
            label="Field Type"
            :options="typeOptions"
            @input="clear"
            map-options
            emit-value
            outlined
            dense
            :disable="editing"
            v-model="localField.type"
            :rules="[val => !!val || '*Required']"
          />
        </q-card-section>
        <!-- input options select for single and multiple input type -->
        <q-card-section v-if="localField.type === 'single' || localField.type == 'multiple'">
          <q-select
            dense
            label="Input Options (press Enter after typing each option)"
            filled
            v-model="localField.options"
            use-input
            use-chips
            multiple
            hide-dropdown-icon
            input-debounce="0"
            new-value-mode="add-unique"
            @input="
              localField.default_value_string = '';
              localField.default_values_multiple = [];
            "
          />
        </q-card-section>
        <!-- default value -->
        <q-card-section v-if="!!localField.type">
          <!-- For datetime field -->
          <q-input
            v-if="localField.type === 'datetime'"
            outlined
            dense
            label="Default Value"
            v-model="localField.default_value_string"
            :rules="[...defaultValueRules]"
            reactive-rules
          >
            <template v-slot:append>
              <q-icon name="event" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-date v-model="localField.default_value_string" mask="YYYY-MM-DD HH:mm">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Close" color="primary" flat />
                    </div>
                  </q-date>
                </q-popup-proxy>
              </q-icon>
              <q-icon name="access_time" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-time v-model="localField.default_value_string" mask="YYYY-MM-DD HH:mm">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Close" color="primary" flat />
                    </div>
                  </q-time>
                </q-popup-proxy>
              </q-icon>
            </template>
          </q-input>

          <!-- For Checkbox -->
          <q-toggle
            v-else-if="localField.type == 'checkbox'"
            label="Default Value"
            v-model="localField.default_value_bool"
            color="green"
          />

          <!-- Dropdown Single -->
          <q-select
            v-else-if="localField.type === 'single'"
            label="Default Value"
            :options="localField.options"
            outlined
            dense
            v-model="localField.default_value_string"
            :rules="[...defaultValueRules]"
            reactive-rules
          />

          <!-- Dropdown Multiple -->
          <q-select
            v-else-if="localField.type === 'multiple'"
            label="Default Value(s)"
            :options="localField.options"
            outlined
            dense
            multiple
            v-model="localField.default_values_multiple"
            :rules="[...defaultValueRules]"
            reactive-rules
          />

          <!-- For everything else -->
          <q-input
            v-else
            label="Default Value"
            :type="localField.type === 'text' ? 'text' : 'number'"
            outlined
            dense
            v-model="localField.default_value_string"
            :rules="[...defaultValueRules]"
            reactive-rules
          />
        </q-card-section>
        <q-card-section>
          <q-toggle
            v-if="localField.type !== 'checkbox'"
            label="Required"
            v-model="localField.required"
            color="green"
          />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn flat label="Submit" color="primary" type="submit" />
        </q-card-actions>
      </q-form>
    </q-card>
  </q-dialog>
</template>

<script>
import mixins from "@/mixins/mixins";

export default {
  name: "CustomFieldsForm",
  mixins: [mixins],
  props: { field: Object, model: String },
  data() {
    return {
      localField: {
        name: "",
        model: "",
        type: "",
        options: [],
        required: false,
        default_value_string: "",
        default_value_bool: false,
        default_values_multiple: [],
      },
      modelOptions: [
        { label: "Client", value: "client" },
        { label: "Site", value: "site" },
        { label: "Agent", value: "agent" },
      ],
      typeOptions: [
        { label: "Text", value: "text" },
        { label: "Number", value: "number" },
        { label: "Dropdown Single", value: "single" },
        { label: "Dropdown Multiple", value: "multiple" },
        { label: "DateTime", value: "datetime" },
        { label: "Checkbox", value: "checkbox" },
      ],
    };
  },
  computed: {
    title() {
      return this.editing ? "Edit Custom Field" : "Add Custom Field";
    },
    editing() {
      return !!this.field;
    },
    defaultValueRules() {
      if (this.localField.required) {
        return [val => !!val || `Default Value needs to be set for required fields`];
      }
    },
  },
  methods: {
    submit() {
      this.$q.loading.show();

      let data = {
        ...this.localField,
      };

      if (this.editing) {
        this.$axios
          .put(`/core/customfields/${data.id}/`, data)
          .then(r => {
            this.$q.loading.hide();
            this.onOk();
            this.notifySuccess("Custom field edited!");
          })
          .catch(e => {
            this.$q.loading.hide();
            this.onOk();
            this.notifyError("There was an error editing the custom field");
          });
      } else {
        this.$axios
          .post("/core/customfields/", data)
          .then(r => {
            this.$q.loading.hide();
            this.onOk();
            this.notifySuccess("Custom field added!");
          })
          .catch(e => {
            this.$q.loading.hide();
            this.notifyError("There was an error adding the custom field");
          });
      }
    },
    clear() {
      this.localField.options = [];
      this.localField.required = false;
      this.localField.default_value_string = "";
      this.localField.default_values_multiple = [];
      this.localField.default_value_bool = false;
    },
    show() {
      this.$refs.dialog.show();
    },
    hide() {
      this.$refs.dialog.hide();
    },
    onHide() {
      this.$emit("hide");
    },
    onOk() {
      this.$emit("ok");
      this.hide();
    },
  },
  mounted() {
    // If pk prop is set that means we are editting
    if (this.field) Object.assign(this.localField, this.field);

    // Set model to current tab
    if (this.model) this.localField.model = this.model;
  },
};
</script>