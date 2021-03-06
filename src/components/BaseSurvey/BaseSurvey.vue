<template>
  <SurveyComponent :survey="model" class="base-survey" />
</template>

<script>
import cloneDeep from 'lodash/cloneDeep';
import * as SurveyVue from 'survey-vue';
import resolveConfig from 'tailwindcss/resolveConfig';
import tailwindConfig from '../../../tailwind.config';
import 'survey-vue/survey.css';

const { theme } = resolveConfig(tailwindConfig);

const SurveyComponent = SurveyVue.Survey;

export const mergeDefaultValues = (baseJson, defaultValues) => {
  const json = cloneDeep(baseJson);
  const elements = json.pages
    ? json.pages.map((page) => page.elements).flat()
    : json.elements;
  Object.entries(defaultValues).forEach(([key, value]) => {
    const matchingField = elements.find((el) => el.name === key);
    if (matchingField) {
      matchingField.defaultValue = value;
    }
  });
  return json;
};

export default {
  components: { SurveyComponent },
  props: {
    survey: {
      type: Object,
      required: true,
    },
    initialValues: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    const json = mergeDefaultValues(this.survey, this.initialValues);
    const model = new SurveyVue.Model(json);
    return { model };
  },
  computed: {
    surveyData() {
      return this.model.data;
    },
  },
  watch: {
    surveyData() {
      this.$emit('input', this.model);
    },
  },
  created() {
    this.model.onComplete.add((sender) => {
      this.$emit('complete', sender);
    });

    const defaultThemeColors = SurveyVue.StylesManager.ThemeColors.default;
    defaultThemeColors['$border-color'] = theme.colors.gray['5'];
    defaultThemeColors['$body-container-background-color'] =
      theme.colors.gray['7'];
    defaultThemeColors['$disable-color'] = theme.colors.black;
    defaultThemeColors['$disabled-label-color'] = theme.colors.gray['3'];
    defaultThemeColors['$disabled-switch-color'] = 'rgba(72, 29, 115, 0.5)';
    defaultThemeColors['$disabled-slider-color'] = theme.colors.white;
    defaultThemeColors['$error-background-color'] = theme.colors['error-light'];
    defaultThemeColors['$error-color'] = theme.colors.error;
    defaultThemeColors['$inputs-background-color'] = theme.colors.white;
    defaultThemeColors['$main-color'] = theme.colors.primary;
    defaultThemeColors['$main-hover-color'] = theme.colors['primary-tint']['1'];
    defaultThemeColors['$slider-color'] = theme.colors.white;
    defaultThemeColors['$text-color'] = theme.colors.black;
    SurveyVue.StylesManager.applyTheme();

    // Other possible theme values to be declared:
    // $header-background-color: "#e7e7e7"
    // $header-color: "#6d7072"
    // $progress-text-color: "#9d9d9d"

    // Advance to the most recently completed page on the survey.
    const initialValueKeys = Object.keys(this.initialValues);
    if (
      initialValueKeys.length &&
      this.survey.pages.length > 1 &&
      !this.model.isCurrentPageHasErrors
    ) {
      let lastCompletedPageName;
      this.survey.pages.forEach((page) => {
        if (
          page.elements.some((element) =>
            initialValueKeys.includes(element.name)
          )
        ) {
          lastCompletedPageName = page.name;
        }
      });
      this.model.currentPage = lastCompletedPageName;
    }
  },
};
</script>

<style>
.base-survey.sv_main {
  @apply font-body-medium font-sans;
}

.base-survey.sv_main .sv_body {
  @apply border-none p-0;
}

.base-survey.sv_main input[type='button'],
.base-survey.sv_main button {
  @apply duration-ludicrous
    ease-out
    font-semibold
    px-4
    h-8
    rounded
    text-14
    transition-all;
}

.base-survey.sv_main textarea:hover {
  @apply border-gray-3;
}

.base-survey.sv_main .sv-boolean__switch {
  @apply duration-light
    ease-out 
    inline-flex 
    items-center 
    mx-2 
    transition-colors 
    w-12;
}

.base-survey.sv_main .sv-boolean--indeterminate .sv-boolean__switch {
  @apply bg-gray-4;
}

.base-survey.sv_main textarea:focus,
.base-survey.sv_main .sv-boolean input:focus ~ .sv-boolean__switch,
.base-survey.sv_main input[type='button']:focus,
.base-survey.sv_main button:focus {
  @apply outline-none shadow-outline;
}

.base-survey.sv_main .sv_container .sv_body .sv_p_root .sv_q textarea {
  @apply appearance-none border-2 font-body-text font-sans p-4 rounded-md;
}

.base-survey.sv_main
  .sv_q_erbox:not([style*='display: none']):not([style*='display:none']) {
  @apply bg-error-light border-none p-4 rounded-md text-error;
}

.base-survey.sv_main .sv-boolean__slider {
  @apply h-4 w-4;
}

.base-survey.sv_main .sv-boolean--indeterminate .sv-boolean__slider {
  margin-left: calc(50% - 8px);
}

.base-survey.sv_main .sv-boolean--checked .sv-boolean__slider {
  margin-left: calc(100% - 16px);
}

.base-survey.sv_main .sv_container .sv_body .sv_p_root .sv_q {
  @apply px-6 py-4;
}

.base-survey.sv_main input[type='radio'] {
  @apply sr-only;
}

.base-survey.sv_main .sv_q_radiogroup_label .circle {
  @apply align-middle
    border
    border-gray-4
    border-solid
    box-border
    h-6 
    inline-flex 
    items-center 
    justify-center 
    mr-4 
    relative 
    rounded-full 
    w-6;
}

.base-survey.sv_main .sv_q_radiogroup_label .circle svg {
  @apply hidden;
}

.base-survey.sv_main .checked .sv_q_radiogroup_label .circle {
  @apply border-2 border-primary;
}

.base-survey.sv_main .checked .sv_q_radiogroup_label .circle::before {
  @apply absolute bg-primary block h-0 rounded-full w-1/2;
  content: '';
  padding-top: 50%;
}
</style>
