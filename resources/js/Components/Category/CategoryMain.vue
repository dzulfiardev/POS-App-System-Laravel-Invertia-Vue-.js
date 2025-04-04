<template>
  <div class="container my-5">
    <div class="md:grid md:grid-cols-3 md:gap-6">
      <!--  -->
      <div class="mt-10 sm:mt-0">
        <div class="md:grid md:grid-cols-3 md:gap-1">
          <div class="mt-5 md:mt-0 md:col-span-3">
            <form @submit.prevent="submit">
              <div class="shadow overflow-hidden sm:rounded-md">
                <div class="px-4 py-5 bg-white sm:p-6">
                  <div class="grid grid-cols-6 gap-6">
                    <div class="col-span-6 sm:col-span-6">
                      <label
                        for="code"
                        class="block text-sm font-medium text-gray-700"
                        >Code</label
                      >
                      <input
                        type="text"
                        name="code"
                        id="code"
                        autocomplete="given-name"
                        class="
                          mt-1
                          focus:ring-indigo-500 focus:border-indigo-500
                          block
                          w-full
                          shadow-sm
                          sm:text-sm
                          border-gray-300
                          rounded-md
                        "
                        placeholder="0"
                        autofocus
                        v-model="form.category_code"
                      />
                      <small
                        class="text-red-600"
                        v-if="form.errors.category_code"
                        >{{ form.errors.category_code }}</small
                      >
                      <small class="text-red-600" v-if="validation.number">{{
                        validation.number
                      }}</small>
                    </div>
                  </div>
                  <div class="grid grid-cols-6 gap-6 mt-2">
                    <div class="col-span-6 sm:col-span-6">
                      <label
                        for="name"
                        class="block text-sm font-medium text-gray-700"
                        >Name</label
                      >
                      <input
                        type="text"
                        name="name"
                        id="name"
                        autocomplete="given-name"
                        class="
                          mt-1
                          focus:ring-indigo-500 focus:border-indigo-500
                          block
                          w-full
                          shadow-sm
                          sm:text-sm
                          border-gray-300
                          rounded-md
                        "
                        v-model="form.category_name"
                      />
                      <small
                        class="text-red-600"
                        v-if="form.errors.category_name"
                        >{{ form.errors.category_name }}</small
                      >
                    </div>
                  </div>
                </div>

                <div class="px-4 py-3 bg-gray-50 text-right sm:px-6">
                  <button
                    type="button"
                    class="
                      inline-flex
                      justify-center
                      py-2
                      px-4
                      mx-1
                      border border-transparent
                      shadow-sm
                      text-sm
                      font-medium
                      rounded-md
                      text-white
                      bg-stone-700
                      hover:bg-stone-800
                      focus:outline-none
                      focus:ring-2
                      focus:ring-offset-2
                      focus:ring-stone-600
                    "
                    :class="{ 'opacity-25': form.processing }"
                    :disabled="form.processing"
                    @click="closeEdit()"
                    v-if="action.cancelUpdate"
                  >
                    Cancel
                  </button>

                  <button
                    type="submit"
                    class="
                      inline-flex
                      justify-center
                      py-2
                      px-4
                      border border-transparent
                      shadow-sm
                      text-sm
                      font-medium
                      rounded-md
                      text-white
                      bg-indigo-600
                      hover:bg-indigo-700
                      focus:outline-none
                      focus:ring-2
                      focus:ring-offset-2
                      focus:ring-indigo-500
                    "
                    :class="{ 'opacity-25': form.processing }"
                    :disabled="form.processing"
                  >
                    Save
                  </button>

                  <progress
                    v-if="form.progress"
                    :value="form.progress.percentage"
                    max="100"
                  >
                    {{ form.progress.percentage }}%
                  </progress>
                </div>
              </div>
            </form>
          </div>
        </div>
      </div>

      <!-- Separator -->

      <div class="mt-5 md:mt-0 md:col-span-2">
        <div class="shadow sm:rounded-md sm:overflow-hidden">
          <div class="px-4 py-2 bg-white space-y-6 sm:p-6">
            <DataTable
              :head="tableHead"
              :values="data"
              :showEntries="showEntries"
              :currentEntries="currentEntries"
              :params="params"
              :checkbox="checkbox"
              @sort="sort"
              @edit="edit"
              @destroy="destroy"
              @bulkDestroy="bulkDestroy"
            />
            <Pagination class="mt-6" :links="data.links" />
          </div>
        </div>
      </div>
    </div>
    <!-- Delete Modal -->
    <TransitionRoot as="template" :show="modal.delete">
      <DeleteModal
        :deleteModal="deleteModal"
        @close="closeDeleteModal"
        @acceptDestroy="acceptDestroy"
      />
    </TransitionRoot>
    <TransitionRoot as="template" :show="modal.bulkDelete">
      <BulkDeleteModal
        @close="closeBulkDeleteModal"
        @bulkDestroy="bulkDestroy"
        @acceptDestroy="acceptBulkDestroy"
      />
    </TransitionRoot>
  </div>
</template>

<script>
import { reactive, ref } from "vue";
import DataTable from "../DataTable/DataTable.vue";
import { useForm } from "@inertiajs/inertia-vue3";
import { Inertia } from "@inertiajs/inertia";
import NProgress from "nprogress";
import Pagination from "../Pagination/Pagination.vue";
import axios from "axios";
import { TransitionRoot } from "@headlessui/vue";
import DeleteModal from "../Modal/DeleteModal.vue";
import BulkDeleteModal from "../Modal/DeleteModal.vue";
import { CheckCircleIcon } from "@heroicons/vue/outline";

export default {
  props: {
    errors: Object,
    data: Object,
    filters: Object,
  },
  components: {
    DataTable,
    Pagination,
    TransitionRoot,
    DeleteModal,
    BulkDeleteModal,
    CheckCircleIcon,
  },
  setup(props) {
    const filters = props.filters;
    const params = reactive({
      search: filters.search,
      field: filters.field,
      direction: filters.direction,
    });

    function sort(field) {
      if (field) {
        params.field = field;
        params.direction = params.direction === "asc" ? "desc" : "asc";
      } else {
        params.field = "";
        params.direction = "";
      }
    }

    const form = useForm({
      id: null,
      category_code: null,
      category_name: null,
      action: null,
    });

    const validation = reactive({ number: "" });

    function submit() {
      if (!/^[0-9]+$/.test(form.category_code)) {
        validation.number =
          "Please only enter numeric characters only for Category Code";
        return false;
      }
      form.post("/category-create", {
        preserveScroll: true,
        onSuccess: () => {
          form.reset();
          validation.number = "";
          action.cancelUpdate = false;
        },
      });
    }

    const action = reactive({
      id: 0,
      cancelUpdate: false,
      flashMessage: false,
      flashMessageText: "",
    });

    function edit(id) {
      form.action = "edit";
      action.cancelUpdate = true;
      NProgress.start();
      axios
        .post("/category-edit", {
          id: id,
        })
        .then((res) => {
          NProgress.done();
          form.id = res.data.id;
          form.category_code = res.data.category_code;
          form.category_name = res.data.category_name;
        })
        .catch((err) => {
          console.log(err);
        });
    }

    function closeEdit() {
      action.cancelUpdate = false;
      form.reset();
    }

    const modal = reactive({
      delete: false,
      bulkDelete: false,
    });

    function destroy(id) {
      action.id = id;
      modal.delete = !modal.delete;
    }

    function acceptDestroy() {
      Inertia.visit("/category-delete", {
        method: "delete",
        data: {
          id: action.id,
        },
        onSuccess: () => {
          action.id = 0;
        },
      });
    }

    function closeDeleteModal() {
      modal.delete = !modal.delete;
    }

    const checkbox = useForm({
      all: false,
      categories: [],
    });

    function bulkDestroy() {
      modal.bulkDelete = !modal.bulkDelete;
    }

    function acceptBulkDestroy() {
      Inertia.visit("/category-bulk-delete", {
        method: "delete",
        data: {
          categoriesId: checkbox.categories,
        },
      });
    }

    function closeBulkDeleteModal() {
      modal.bulkDelete = !modal.bulkDelete;
    }

    return {
      form,
      submit,
      validation,
      params,
      sort,
      action,
      edit,
      destroy,
      modal,
      closeDeleteModal,
      closeEdit,
      checkbox,
      acceptDestroy,
      bulkDestroy,
      acceptBulkDestroy,
      closeBulkDeleteModal,
    };
  },
  watch: {
    params: {
      handler() {
        let params = this.params;
        // let params = pickBy(this.params);

        Object.keys(params).forEach((key) => {
          if (params[key] == "") {
            delete params[key];
          }
        });

        this.$inertia.get(this.route("category"), params, {
          replace: true,
          preserveState: true,
        });
      },
      deep: true,
    },
  },
};
</script>
