<template>
  <aside v-if="showCookieComply" class="cookie-comply">
    <div class="cookie-comply__header">
      <slot name="header">
        <h3 class="cookie-comply__header-title">{{ headerTitle }}</h3>
        <p class="cookie-comply__header-description">{{ headerDescription }}</p>
      </slot>
    </div>

    <div class="cookie-comply__actions">
      <cookie-comply-button @handleClick="rejectCookies">
        {{ rejectLabel }}
      </cookie-comply-button>
      <cookie-comply-button
        class-name="cookie-comply__button-accept"
        @handleClick="handleAcceptAll"
      >
        {{ acceptAllLabel }}
      </cookie-comply-button>
    </div>

    <Teleport :to="target">
      <cookie-comply-modal
        v-if="isModalOpen"
        :preferences="preferences"
        @cookie-comply-save="onSave"
        @cookie-comply-close="isModalOpen = false"
      >
        <template #modal-header>
          <slot name="modal-header"></slot>
        </template>

        <template #modal-body="{ preference, index }">
          <slot
            name="modal-body"
            :preference="preference"
            :index="index"
          ></slot>
        </template>

        <template #modal-footer>
          <slot name="modal-footer"></slot>
        </template>
      </cookie-comply-modal>
    </Teleport>
  </aside>
</template>

<script>
import CookieComplyModal from './CookieComplyModal.vue';
import CookieComplyButton from './CookieComplyButton.vue';

export default {
  name: 'CookieComply',
  components: { CookieComplyModal, CookieComplyButton },
  props: {
    headerTitle: { type: String, default: 'Cookie settings' },
    headerDescription: {
      type: String,
      default:
        'We use cookies and similar technologies to help personalize content and offer a better experience. You can opt to customize them by clicking the preferences button.',
    },
    rejectLabel: { type: String, default: 'Reject' },
    acceptAllLabel: { type: String, default: 'Accept All' },
    preferences: { type: Array, default: () => [] },
    target: { type: String, default: 'body' }
  },
  emits: ['on-accept-all-cookies', 'on-save-cookie-preferences','on-declined'],
  data() {
    return {
      showCookieComply: true,
      isModalOpen: false,
    };
  },
  mounted() {
    if (localStorage.getItem('cookie-comply')) {
      this.showCookieComply = false;
    }
  },
  methods: {
    handleAcceptAll() {
      this.showCookieComply = false;
      localStorage.setItem('cookie-comply', 'all');
      this.$emit('on-accept-all-cookies');
    },
    openPreferences() {
      this.isModalOpen = true;
    },
    rejectCookies() {
      localStorage.setItem('cookie-comply', 'rejected');
      this.$emit('on-reject');
    },
    onSave(data) {
      this.isModalOpen = false;
      this.showCookieComply = false;

      // transform Proxy into array of selected preferences
      const preferencesArray = Object.values(data);

      localStorage.setItem('cookie-comply', JSON.stringify(preferencesArray));
      this.$emit('on-save-cookie-preferences', preferencesArray);
    },
  },
};
</script>

<style>
@import '../styles/variables.css';

.cookie-comply {
  display: grid;
  grid-gap: var(--spacing-lg);
  grid-template-columns: 1fr minmax(35%, 40%);
  position: absolute;
  bottom: var(--spacing-sm);
  left: var(--spacing-sm);
  right: var(--spacing-sm);
  background-color: var(--color-white);
  box-shadow: var(--box-shadow);
  padding: var(--spacing-md);
  border-radius: var(--border-radius);
}

@media (max-width: 1024px) {
  .cookie-comply {
    grid-template-columns: none;
  }
}

.cookie-comply__header {
  justify-self: flex-start;
  text-align: initial;
}

.cookie-comply__header-title,
.cookie-comply__header-description {
  margin: 0;
}
.cookie-comply__header-title {
  margin-bottom: var(--spacing-sm);
}
.cookie-comply__header-description {
  line-height: 20px;
}

.cookie-comply__actions {
  display: grid;
  grid-gap: var(--spacing-lg);
  grid-template-columns: repeat(2, 1fr);
  align-self: center;
}

@media (max-width: 480px) {
  .cookie-comply__header {
    margin-bottom: var(--spacing-sm);
  }

  .cookie-comply__actions {
    grid-template-columns: auto;
  }
}

.cookie-comply__button-accept {
  background-color: var(--color-green);
  color: var(--color-white);
  border: none;
}
</style>
