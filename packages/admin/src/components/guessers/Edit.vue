<template>
  <va-edit-layout :title="title">
    <va-form :id="id" :item="item">
      <v-row justify="center">
        <v-col sm="4">
          <v-card>
            <v-card-text>
              <div>
                <component
                  v-for="input in inputs"
                  :key="input.source"
                  :source="input.source"
                  :is="`va-${input.type}-input`"
                ></component>
              </div>
              <va-save-button></va-save-button>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </va-form>
  </va-edit-layout>
</template>

<script>
import { guessInputs } from "../../utils/guesser";

export default {
  props: ["id", "title", "resource", "item"],
  data() {
    return {
      inputs: [],
    };
  },
  async created() {
    this.inputs = await guessInputs(
      this.$store,
      this.$i18n,
      this.resource.name
    );

    if (this.$guessLogger) {
      this.$guessLogger(this.resource.name, "edit", this.inputs);
    }
  },
};
</script>
