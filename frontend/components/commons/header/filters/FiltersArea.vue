<!--
  - coding=utf-8
  - Copyright 2021-present, the Recognai S.L. team.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <div class="filters__area">
    <div class="filters__content">
      <div class="container">
        <div class="filters__row">
          <div class="filters__block">
            <SearchBar
              class="filters__searchbar"
              :dataset="dataset"
              @submit="onTextQuerySearch"
            />
            <FiltersList
              :dataset="dataset"
              @applyFilter="onApplyFilter"
              @applyMetaFilter="onApplyMetaFilter"
              @applySortBy="onApplySortBy"
              @removeAllMetadataFilters="onRemoveAllMetadataFilters"
              @removeFiltersByGroup="onRemoveFiltersByGroup"
            ></FiltersList>
          </div>
          <slot />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapActions } from "vuex";
export default {
  props: {
    dataset: {
      type: Object,
      default: () => ({}),
    },
  },
  data: () => ({
    sortable: {
      type: Boolean,
      default: false,
    },
    sortBy: "gold",
    sortByDir: "desc",
    sortOptions: [
      { filter: "annotated_as", text: "Annotated as", range: ["A", "Z"] },
      { filter: "predicted_as", text: "Predicted as", range: ["A", "Z"] },
      { filter: "score", text: "Score", range: ["0", "1"] },
    ],
  }),
  computed: {
    viewMode() {
      return this.dataset.viewSettings.viewMode;
    },
  },
  methods: {
    ...mapActions({
      search: "entities/datasets/search",
    }),
    onTextQuerySearch(text) {
      if (text === "") {
        text = undefined;
      }
      this.search({ dataset: this.dataset, query: { text } });
    },
    onApplyFilter({ filter, values }) {
      if (Array.isArray(values) && !values.length) {
        values = undefined;
      }
      this.search({ dataset: this.dataset, query: { [filter]: values } });
    },
    onApplyMetaFilter({ filter, values }) {
      this.search({
        dataset: this.dataset,
        query: { metadata: { [filter]: values } },
      });
    },
    async onRemoveAllMetadataFilters(filters) {
      let query = {};
      filters.forEach((f) => (query[f.key] = []));
      await this.search({ dataset: this.dataset, query: { metadata: query } });
    },
    async onRemoveFiltersByGroup(filters) {
      let query = {};
      filters.forEach(
        (f) => (query[f.key] = f.key === "score" ? undefined : [])
      );
      await this.search({ dataset: this.dataset, query: query });
    },
    async onApplySortBy(sortList) {
      await this.search({
        dataset: this.dataset,
        query: this.dataset.query,
        sort: sortList,
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  @extend %container;
  padding-top: 0;
  padding-bottom: 0;
  margin-left: 0;
  @extend %collapsable-if-metrics !optional;
}

.filters {
  &__area {
    display: flex;
    align-items: center;
    top: -1em;
    left: 0;
    right: 0;
  }
  &__row {
    display: flex;
    align-items: center;
  }
  &__content {
    padding: 32px 0;
    position: relative;
    width: 100%;
  }
  &__block {
    display: flex;
    align-items: center;
    width: calc(100% - 300px);
  }
  &__searchbar {
    margin-right: 10px;
    width: 100%;
    @include media(">desktop") {
      margin-right: 15px;
    }
    &.--extended {
      width: 100%;
      margin-right: 0;
    }
  }
  &--disabled {
    ::v-deep * {
      pointer-events: none !important;
      cursor: pointer;
    }
    ::v-deep .filters__searchbar {
      opacity: 0.4;
    }
    ::v-deep .filters--sort {
      align-items: center;
      opacity: 0.4;
    }
  }
}
</style>
