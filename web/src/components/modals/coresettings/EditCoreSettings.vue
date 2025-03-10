<template>
  <q-card style="min-width: 85vh">
    <q-splitter v-model="splitterModel">
      <template v-slot:before>
        <q-tabs dense v-model="tab" vertical class="text-primary">
          <q-tab name="general" label="General" />
          <q-tab name="emailalerts" label="Email Alerts" />
          <q-tab name="smsalerts" label="SMS Alerts" />
          <q-tab name="meshcentral" label="MeshCentral" />
          <q-tab name="customfields" label="Custom Fields" />
        </q-tabs>
      </template>
      <template v-slot:after>
        <q-form @submit.prevent="editSettings">
          <q-card-section class="row items-center">
            <div class="text-h6">Global Settings</div>
            <q-space />
            <q-btn icon="close" flat round dense v-close-popup />
          </q-card-section>
          <q-scroll-area :thumb-style="thumbStyle" style="height: 60vh">
            <q-tab-panels v-model="tab" animated transition-prev="jump-up" transition-next="jump-up">
              <!-- general -->
              <q-tab-panel name="general">
                <div class="text-subtitle2">General</div>
                <hr />
                <q-card-section class="row">
                  <q-checkbox v-model="settings.agent_auto_update" label="Enable agent automatic self update" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Default agent timezone:</div>
                  <div class="col-2"></div>
                  <q-select
                    outlined
                    dense
                    options-dense
                    v-model="settings.default_time_zone"
                    :options="allTimezones"
                    class="col-6"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Default server policy:</div>
                  <div class="col-2"></div>
                  <q-select
                    clearable
                    map-options
                    emit-value
                    outlined
                    dense
                    options-dense
                    v-model="settings.server_policy"
                    :options="policies"
                    class="col-6"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Default workstation policy:</div>
                  <div class="col-2"></div>
                  <q-select
                    clearable
                    map-options
                    emit-value
                    outlined
                    dense
                    options-dense
                    v-model="settings.workstation_policy"
                    :options="policies"
                    class="col-6"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Default alert template:</div>
                  <div class="col-2"></div>
                  <q-select
                    clearable
                    map-options
                    emit-value
                    outlined
                    dense
                    options-dense
                    v-model="settings.alert_template"
                    :options="alertTemplateOptions"
                    class="col-6"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Remove Check History older than (days):</div>
                  <div class="col-2"></div>
                  <q-input outlined dense v-model="settings.check_history_prune_days" class="col-6" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Reset Patch Policy on Agents:</div>
                  <div class="col-2"></div>
                  <q-btn color="negative" label="Reset" @click="showResetPatchPolicy" />
                </q-card-section>
              </q-tab-panel>
              <!-- email alerts -->
              <q-tab-panel name="emailalerts">
                <div class="text-subtitle2 row">
                  <div>Email Alert Routing</div>
                  <q-space />
                  <div>
                    <q-btn
                      size="sm"
                      color="grey-5"
                      icon="fas fa-plus"
                      text-color="black"
                      label="Add emails"
                      @click="toggleAddEmail"
                    />
                  </div>
                </div>
                <hr />
                <q-card-section class="row">
                  <div class="col-3">Recipients</div>
                  <div class="col-4"></div>
                  <div class="col-5">
                    <q-list dense v-if="ready && settings.email_alert_recipients.length !== 0">
                      <q-item
                        v-for="email in settings.email_alert_recipients"
                        :key="email"
                        clickable
                        v-ripple
                        @click="removeEmail(email)"
                      >
                        <q-item-section>
                          <q-item-label>{{ email }}</q-item-label>
                        </q-item-section>
                        <q-item-section side>
                          <q-icon name="delete" color="red" />
                        </q-item-section>
                      </q-item>
                    </q-list>
                    <q-list v-else>
                      <q-item-section>
                        <q-item-label>No recipients</q-item-label>
                      </q-item-section>
                    </q-list>
                  </div>
                </q-card-section>
                <!-- smtp -->
                <div class="text-subtitle2">SMTP Settings</div>
                <hr />
                <q-card-section class="row">
                  <div class="col-2">From:</div>
                  <div class="col-4"></div>
                  <q-input
                    outlined
                    dense
                    v-model="settings.smtp_from_email"
                    class="col-6 q-pa-none"
                    :rules="[val => isValidEmail(val) || 'Invalid email']"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-2">Host:</div>
                  <div class="col-4"></div>
                  <q-input outlined dense v-model="settings.smtp_host" class="col-6 q-pa-none" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-2">Port:</div>
                  <div class="col-4"></div>
                  <q-input
                    dense
                    v-model.number="settings.smtp_port"
                    type="number"
                    filled
                    class="q-pa-none"
                    :rules="[val => (val > 0 && val <= 65535) || 'Invalid Port']"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <q-checkbox
                    v-model="settings.smtp_requires_auth"
                    label="My Server Requires Authentication"
                    class="q-pa-none"
                  />
                </q-card-section>
                <q-card-section class="row" v-show="settings.smtp_requires_auth">
                  <div class="col-2">Username:</div>
                  <div class="col-4"></div>
                  <q-input outlined dense v-model="settings.smtp_host_user" class="col-6 q-pa-none" />
                </q-card-section>
                <q-card-section class="row" v-show="settings.smtp_requires_auth">
                  <div class="col-2">Password:</div>
                  <div class="col-4"></div>
                  <q-input
                    outlined
                    dense
                    class="col-6 q-pa-none"
                    v-model="settings.smtp_host_password"
                    :type="isPwd ? 'password' : 'text'"
                  >
                    <template v-slot:append>
                      <q-icon
                        :name="isPwd ? 'visibility_off' : 'visibility'"
                        class="cursor-pointer"
                        @click="isPwd = !isPwd"
                      />
                    </template>
                  </q-input>
                </q-card-section>
              </q-tab-panel>
              <!-- twilio sms alerts -->
              <q-tab-panel name="smsalerts">
                <div class="text-subtitle2 row">
                  <div>SMS Alert Routing</div>
                  <q-space />
                  <div>
                    <q-btn
                      size="sm"
                      color="grey-5"
                      icon="fas fa-plus"
                      text-color="black"
                      label="Add numbers"
                      @click="toggleAddSMSNumber"
                    />
                  </div>
                </div>
                <hr />
                <q-card-section class="row">
                  <div class="col-3">Recipients</div>
                  <div class="col-4"></div>
                  <div class="col-5">
                    <q-list dense v-if="ready && settings.sms_alert_recipients.length !== 0">
                      <q-item
                        v-for="num in settings.sms_alert_recipients"
                        :key="num"
                        clickable
                        v-ripple
                        @click="removeSMSNumber(num)"
                      >
                        <q-item-section>
                          <q-item-label>{{ num }}</q-item-label>
                        </q-item-section>
                        <q-item-section side>
                          <q-icon name="delete" color="red" />
                        </q-item-section>
                      </q-item>
                    </q-list>
                    <q-list v-else>
                      <q-item-section>
                        <q-item-label>No recipients</q-item-label>
                      </q-item-section>
                    </q-list>
                  </div>
                </q-card-section>
                <!-- smtp -->
                <div class="text-subtitle2">Twilio Settings</div>
                <hr />
                <q-card-section class="row">
                  <div class="col-3">Twilio Number:</div>
                  <div class="col-3"></div>
                  <q-input
                    outlined
                    dense
                    v-model="settings.twilio_number"
                    class="col-6 q-pa-none"
                    placeholder="+12131231234"
                  />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-3">Twilio Account SID:</div>
                  <div class="col-3"></div>
                  <q-input outlined dense v-model="settings.twilio_account_sid" class="col-6 q-pa-none" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-3">Twilio Auth Token:</div>
                  <div class="col-3"></div>
                  <q-input outlined dense v-model="settings.twilio_auth_token" class="col-6 q-pa-none" />
                </q-card-section>
              </q-tab-panel>
              <!-- meshcentral -->
              <q-tab-panel name="meshcentral">
                <div class="text-subtitle2">MeshCentral Settings</div>
                <hr />
                <q-card-section class="row">
                  <div class="col-4">Username:</div>
                  <div class="col-2"></div>
                  <q-input dense filled v-model="settings.mesh_username" class="col-6" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Mesh Site:</div>
                  <div class="col-2"></div>
                  <q-input dense filled v-model="settings.mesh_site" class="col-6" />
                </q-card-section>
                <q-card-section class="row">
                  <div class="col-4">Mesh Token:</div>
                  <div class="col-2"></div>
                  <q-input dense filled v-model="settings.mesh_token" class="col-6" />
                </q-card-section>
              </q-tab-panel>
              <q-tab-panel name="customfields">
                <CustomFields />
              </q-tab-panel>
            </q-tab-panels>
          </q-scroll-area>
          <q-card-section class="row items-center">
            <q-btn v-show="tab !== 'customfields'" label="Save" color="primary" type="submit" />
            <q-btn
              v-show="tab === 'emailalerts'"
              label="Save and Test"
              color="primary"
              type="submit"
              class="q-ml-md"
              @click="emailTest = true"
            />
          </q-card-section>
        </q-form>
      </template>
    </q-splitter>
  </q-card>
</template>

<script>
import mixins from "@/mixins/mixins";
import ResetPatchPolicy from "@/components/modals/coresettings/ResetPatchPolicy";
import CustomFields from "@/components/modals/coresettings/CustomFields";

export default {
  name: "EditCoreSettings",
  components: {
    CustomFields,
  },
  mixins: [mixins],
  data() {
    return {
      ready: false,
      policies: [],
      settings: {},
      email: null,
      tab: "general",
      splitterModel: 20,
      isPwd: true,
      allTimezones: [],
      emailTest: false,
      thumbStyle: {
        right: "2px",
        borderRadius: "5px",
        backgroundColor: "#027be3",
        width: "5px",
        opacity: 0.75,
      },
      alertTemplateOptions: [],
    };
  },
  methods: {
    getCoreSettings() {
      this.$axios.get("/core/getcoresettings/").then(r => {
        this.settings = r.data;
        this.allTimezones = Object.freeze(r.data.all_timezones);
        this.ready = true;
      });
    },
    getPolicies() {
      this.$q.loading.show();
      this.$axios
        .get("/automation/policies/")
        .then(r => {
          this.policies = r.data.map(policy => ({ label: policy.name, value: policy.id }));
          this.$q.loading.hide();
        })
        .catch(e => {
          this.$q.loading.hide();
          this.notifyError("Unable to get policies");
        });
    },
    getAlertTemplates() {
      this.$axios.get("alerts/alerttemplates").then(r => {
        this.alertTemplateOptions = r.data.map(template => ({ label: template.name, value: template.id }));
      });
    },
    showResetPatchPolicy() {
      this.$q.dialog({
        component: ResetPatchPolicy,
        parent: this,
      });
    },
    toggleAddEmail() {
      this.$q
        .dialog({
          title: "Add email",
          prompt: {
            model: "",
            isValid: val => this.isValidEmail(val),
            type: "email",
          },
          cancel: true,
          ok: { label: "Add", color: "primary" },
          persistent: false,
        })
        .onOk(data => {
          this.settings.email_alert_recipients.push(data);
        });
    },
    toggleAddSMSNumber() {
      this.$q
        .dialog({
          title: "Add number",
          message:
            "Use E.164 format: must have the <b>+</b> symbol and <span class='text-red'>country code</span>, followed by the <span class='text-green'>phone number</span> e.g. <b>+<span class='text-red'>1</span><span class='text-green'>2131231234</span></b>",
          prompt: {
            model: "",
          },
          html: true,
          cancel: true,
          ok: { label: "Add", color: "primary" },
          persistent: false,
        })
        .onOk(data => {
          this.settings.sms_alert_recipients.push(data);
        });
    },
    removeEmail(email) {
      const removed = this.settings.email_alert_recipients.filter(k => k !== email);
      this.settings.email_alert_recipients = removed;
    },
    removeSMSNumber(num) {
      const removed = this.settings.sms_alert_recipients.filter(k => k !== num);
      this.settings.sms_alert_recipients = removed;
    },
    editSettings() {
      this.$q.loading.show();
      delete this.settings.all_timezones;
      this.$axios
        .patch("/core/editsettings/", this.settings)
        .then(r => {
          this.$q.loading.hide();
          if (this.emailTest) {
            this.$q.loading.show({ message: "Sending test email..." });
            this.$axios
              .get("/core/emailtest/")
              .then(r => {
                this.emailTest = false;
                this.$q.loading.hide();
                this.getCoreSettings();
                this.notifySuccess(r.data, 3000);
              })
              .catch(e => {
                this.emailTest = false;
                this.$q.loading.hide();
                this.notifyError(e.response.data, 7000);
              });
          } else {
            this.$emit("close");
            this.notifySuccess("Settings were edited!");
          }
        })
        .catch(() => {
          this.$q.loading.hide();
          this.notifyError("You have some invalid input. Please check all fields");
        });
    },
  },
  created() {
    this.getCoreSettings();
    this.getPolicies();
    this.getAlertTemplates();
  },
};
</script>