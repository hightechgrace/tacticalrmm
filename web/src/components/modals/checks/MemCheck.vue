<template>
  <q-card style="min-width: 400px">
    <q-card-section class="row items-center">
      <div v-if="mode === 'add'" class="text-h6">Add Memory Check</div>
      <div v-else-if="mode === 'edit'" class="text-h6">Edit Memory Check</div>
      <q-space />
      <q-btn icon="close" flat round dense v-close-popup />
    </q-card-section>

    <q-form @submit.prevent="mode === 'add' ? addCheck() : editCheck()">
      <q-card-section>
        <q-input
          outlined
          type="number"
          v-model.number="memcheck.warning_threshold"
          label="Warning Threshold (%)"
          :rules="[val => val >= 0 || 'Minimum threshold is 0', val => val < 100 || 'Maximum threshold is 99']"
        />
      </q-card-section>
      <q-card-section>
        <q-input
          outlined
          type="number"
          v-model.number="memcheck.error_threshold"
          label="Error Threshold (%)"
          :rules="[val => val >= 0 || 'Minimum threshold is 0', val => val < 100 || 'Maximum threshold is 99']"
        />
      </q-card-section>
      <q-card-section>
        <q-select
          outlined
          dense
          options-dense
          v-model="memcheck.fails_b4_alert"
          :options="failOptions"
          label="Number of consecutive failures before alert"
        />
      </q-card-section>
      <q-card-section>
        <q-input
          outlined
          dense
          type="number"
          v-model.number="memcheck.run_interval"
          label="Check run interval (seconds)"
          hint="Setting this will override the check run interval on the agent"
        />
      </q-card-section>
      <q-card-actions align="right">
        <q-btn v-if="mode === 'add'" label="Add" color="primary" type="submit" />
        <q-btn v-else-if="mode === 'edit'" label="Edit" color="primary" type="submit" />
        <q-btn label="Cancel" v-close-popup />
      </q-card-actions>
    </q-form>
  </q-card>
</template>

<script>
import axios from "axios";
import mixins from "@/mixins/mixins";
export default {
  name: "MemCheck",
  props: {
    agentpk: Number,
    policypk: Number,
    mode: String,
    checkpk: Number,
  },
  mixins: [mixins],
  data() {
    return {
      memcheck: {
        check_type: "memory",
        warning_threshold: 70,
        error_threshold: 85,
        fails_b4_alert: 1,
        run_interval: 0,
      },
      failOptions: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    };
  },
  methods: {
    getCheck() {
      axios.get(`/checks/${this.checkpk}/check/`).then(r => (this.memcheck = r.data));
    },
    addCheck() {
      if (!this.isValidThreshold(this.memcheck.warning_threshold, this.memcheck.error_threshold)) {
        return;
      }

      const pk = this.policypk ? { policy: this.policypk } : { pk: this.agentpk };
      const data = {
        ...pk,
        check: this.memcheck,
      };
      axios
        .post("/checks/checks/", data)
        .then(r => {
          this.$emit("close");
          this.reloadChecks();
          this.notifySuccess(r.data);
        })
        .catch(e => this.notifyError(e.response.data.non_field_errors));
    },
    editCheck() {
      if (!this.isValidThreshold(this.memcheck.warning_threshold, this.memcheck.error_threshold)) {
        return;
      }

      axios
        .patch(`/checks/${this.checkpk}/check/`, this.memcheck)
        .then(r => {
          this.$emit("close");
          this.reloadChecks();
          this.notifySuccess(r.data);
        })
        .catch(e => this.notifyError(e.response.data.non_field_errors));
    },
    reloadChecks() {
      if (this.agentpk) {
        this.$store.dispatch("loadChecks", this.agentpk);
      }
    },
  },
  created() {
    if (this.mode === "edit") {
      this.getCheck();
    }
  },
};
</script>