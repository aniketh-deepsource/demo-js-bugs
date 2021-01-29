<template>
  <ValidationObserver v-slot="{ invalid }">
    <section class="form">
      <div class="form--heading text-center mb-3">Create a new workspace</div>
      <div class="form--sub-heading text-center mb-3">
        A workspace on Spiti can belong to you or your entire team.
      </div>
      <div class="py-6">
        <div class="mb-6">
          <label class="form--label" for="workspace-name"> Workspace name </label>
          <ValidationProvider v-slot="{ errors }" rules="required|min:3|max:32" mode="eager">
            <input
              autocomplete="off"
              id="workspace-name"
              class="form--input"
              v-model="name"
              type="text"
            />
            <span class="form--error">{{ errors[0] }}</span>
          </ValidationProvider>
        </div>
        <div class="mb-6">
          <label class="form--label" for="workspace-shortcode"> Workspace URL </label>
          <ValidationProvider
            v-slot="{ errors }"
            rules="required|min:3|max:32|unique"
            :debounce="500"
          >
            <div class="form--input flex flex-row items-center">
              <span class="text-gray-500 font-light">spiti.xyz/</span>
              <input
                autocomplete="off"
                id="workspace-shortcode"
                class="apprearance-none bg-offBlack outline-none pl-px font-light w-full"
                v-model="shortcode"
                type="text"
              />
            </div>
            <span class="form--error">{{ errors[0] }}</span>
          </ValidationProvider>
        </div>
        <div class="flex flex-row justify-center py-2">
          <button
            class="btn-primary"
            :disabled="invalid || loading > 0"
            :class="{ 'btn-disabled': invalid || loading > 0 }"
            @click.prevent="create()"
          >
            <span v-if="invalid">Complete the form to continue&hellip;</span>
            <span v-else>
              <span v-if="loading > 0">Creating workspace&hellip;</span>
              <span v-else
                >Create workspace <arrow-right-icon class="h-5 inline-block"></arrow-right-icon
              ></span>
            </span>
          </button>
        </div>
      </div>
    </section>
  </ValidationObserver>
</template>

<script>
import { mapGetters } from 'vuex'
import { ArrowRightIcon } from 'vue-feather-icons'
import { extend, ValidationProvider, ValidationObserver } from 'vee-validate'
import { required, min, max } from 'vee-validate/dist/rules'
import { FUNCTIONS } from '.'
extend('required', {
  ...required,
  message: 'This field is required.'
})
extend('min', {
  ...min,
  message: 'This field should be at least 3 characters.'
})
extend('max', {
  ...max,
  message: 'This field should not be more than 32 characters.'
})
export default {
  name: 'create-workspace',
  components: {
    ArrowRightIcon,
    ValidationProvider,
    ValidationObserver
  },
  data() {
    return {
      name: '',
      shortcode: '',
      loading: 0
    }
  },
  methods: {
    async isWorkspaceShortcodeAvailable() {
      /**
       * Return true if the workspace shortcode is available by querying the database
       */
      const resp = await this.$fire.functions.httpsCallable(
        FUNCTIONS.isWorkspaceShortcodeAvailable
      )({ shortcode: this.shortcode })
      return resp.data.isAvailable
    },
    async create() {
      /**
       * Create a new workspace
       */
      this.loading += 1
      await this.$fire.functions.httpsCallable(FUNCTIONS.createNewWorkspace)({
        name: this.name,
        shortcode: this.shortcode,
        user: this.viewer
      })
      this.loading -= 1
      this.$toast.success(`New Spiti workspace created: ${this.name}`)
      this.$router.push({ name: 'home', params: { slug: this.shortcode } })
    }
  },
  computed: {
    ...mapGetters({
      viewer: 'auth/viewer',
      workspaces: 'workspace/workspaces'
    })
  },
  mounted() {
    extend('unique', {
      validate: this.isWorkspaceShortcodeAvailable,
      message: 'The shortcode is already taken.'
    })
  }
}
</script>
