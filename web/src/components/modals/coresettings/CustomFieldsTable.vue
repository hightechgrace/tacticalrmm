<template>
  <q-table
    dense
    :data="data"
    :columns="columns"
    :pagination.sync="pagination"
    row-key="id"
    binary-state-sort
    hide-pagination
    virtual-scroll
    :rows-per-page-options="[0]"
    no-data-label="No Custom Fields"
  >
    <!-- body slots -->
    <template v-slot:body="props">
      <q-tr
        :props="props"
        class="cursor-pointer"
        @contextmenu="selectedTemplate = props.row"
        @dblclick="editCustomField(props.row)"
      >
        <!-- context menu -->
        <q-menu context-menu>
          <q-list dense style="min-width: 200px">
            <q-item clickable v-close-popup @click="editCustomField(props.row)">
              <q-item-section side>
                <q-icon name="edit" />
              </q-item-section>
              <q-item-section>Edit</q-item-section>
            </q-item>
            <q-item clickable v-close-popup @click="deleteCustomField(props.row)">
              <q-item-section side>
                <q-icon name="delete" />
              </q-item-section>
              <q-item-section>Delete</q-item-section>
            </q-item>

            <q-separator></q-separator>

            <q-item clickable v-close-popup>
              <q-item-section>Close</q-item-section>
            </q-item>
          </q-list>
        </q-menu>
        <!-- name -->
        <q-td>
          {{ props.row.name }}
        </q-td>
        <!-- type -->
        <q-td>
          {{ capitalize(props.row.type) }}
        </q-td>
        <!-- default value -->
        <q-td v-if="props.row.type === 'checkbox'">
          {{ props.row.default_value_bool }}
        </q-td>
        <q-td v-else-if="props.row.type === 'multiple'">
          <span v-if="props.row.default_values_multiple.length > 0">{{ props.row.default_values_multiple }}</span>
        </q-td>
        <q-td v-else>
          {{ props.row.default_value_string }}
        </q-td>
        <!-- required -->
        <q-td>
          <q-icon v-if="props.row.required" name="check" />
        </q-td>
      </q-tr>
    </template>
  </q-table>
</template>

<script>
import CustomFieldsForm from "@/components/modals/coresettings/CustomFieldsForm";
import mixins from "@/mixins/mixins";

export default {
  name: "CustomFieldsTable",
  mixins: [mixins],
  props: {
    data: !Array,
  },
  data() {
    return {
      pagination: {
        rowsPerPage: 0,
        sortBy: "name",
        descending: true,
      },
      columns: [
        {
          name: "name",
          label: "Name",
          field: "name",
          align: "left",
          sortable: true,
        },
        {
          name: "type",
          label: "Field Type",
          field: "type",
          align: "left",
          sortable: true,
        },
        { name: "default_value", label: "Default Value", field: "default_value", align: "left", sortable: true },
        { name: "required", label: "Required", field: "required", align: "left", sortable: true },
      ],
    };
  },
  methods: {
    editCustomField(field) {
      this.$q
        .dialog({
          component: CustomFieldsForm,
          parent: this,
          field: field,
        })
        .onOk(() => {
          this.refresh();
        });
    },
    deleteCustomField(field) {
      this.$q
        .dialog({
          title: `Delete custom field ${field.name}?`,
          cancel: true,
          ok: { label: "Delete", color: "negative" },
        })
        .onOk(() => {
          this.$q.loading.show();
          this.$axios
            .delete(`core/customfields/${field.id}/`)
            .then(r => {
              this.refresh();
              this.$q.loading.hide();
              this.notifySuccess(`Custom Field ${field.name} was deleted!`);
            })
            .catch(error => {
              this.$q.loading.hide();
              this.notifyError(`An Error occured while deleting custom field: ${field.name}`);
            });
        });
    },
    refresh() {
      this.$emit("refresh");
    },
  },
};
</script>