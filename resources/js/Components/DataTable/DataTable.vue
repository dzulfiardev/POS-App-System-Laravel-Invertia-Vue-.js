<template>
  <div class="flex flex-col">
    <div class="overflow-x-auto sm:-mx-6 lg:-mx-8">
      <div class="align-middle inline-block min-w-full sm:px-6 lg:px-8">
        <div class="grid gap-2 grid-cols-1 md:grid-cols-3 my-2 px-1">
          <div>
            <transition name="slide-fade">
              <button
                class="
                  hidden
                  sm:inline-flex
                  justify-center
                  py-2
                  px-2
                  border border-transparent
                  shadow-sm
                  text-sm
                  font-medium
                  rounded-md
                  text-white
                  bg-red-400
                  hover:bg-red-500
                  focus:outline-none
                  focus:ring-2
                  focus:ring-offset-2
                  focus:ring-red-300
                  transition
                  duration-350
                  ease-out
                "
                v-if="checkbox.categories.length > 0"
                @click="$emit('bulkDestroy')"
              >
                <TrashIcon class="h-4 w-4" />
                Selected Delete
              </button>
            </transition>
          </div>
          <div></div>
          <div>
            <input
              type="search"
              aria-label="Search"
              placeholder="Search..."
              class="block w-full rounded-md border-gray-300 shadow-sm"
              v-model="params.search"
            />
            <transition name="slide-fade">
              <button
                class="
                  sm:hidden
                  inline-flex
                  justify-center
                  py-1
                  px-2
                  my-2
                  border border-transparent
                  shadow-sm
                  text-sm
                  font-medium
                  rounded-md
                  text-white
                  bg-red-400
                  hover:bg-red-500
                  focus:outline-none
                  focus:ring-2
                  focus:ring-offset-2
                  focus:ring-red-300
                  transition
                  duration-350
                  ease-out
                "
                v-if="checkbox.categories.length > 0"
                @click="$emit('bulkDestroy')"
              >
                <TrashIcon class="h-4 w-4" />
                Selected Delete
              </button>
            </transition>
          </div>
        </div>
        <div
          class="shadow overflow-hidden border border-gray-200 sm:rounded-lg"
        >
          <table id="data-table" class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr>
                <th
                  scope="col"
                  class="
                    py-3
                    px-6
                    text-left text-xs
                    font-medium
                    text-gray-500
                    uppercase
                    tracking-wider
                  "
                >
                  <input
                    type="checkbox"
                    class="
                      aapperance-none
                      checked:bg-blue-500
                      indeterminate:bg-gray-300
                      cursor-pointer
                      default:ring-2
                      hover:bg-blue-500
                    "
                    v-model="checkbox.all"
                    @click="selectAll()"
                  />
                </th>
                <th
                  scope="col"
                  class="
                    py-3
                    px-6
                    text-left text-xs
                    font-medium
                    text-gray-500
                    uppercase
                    tracking-wider
                  "
                >
                  <div class="flex justify-between">
                    <div
                      class="w-full"
                      style="cursor: pointer"
                      @click="$emit('sort', 'category_code')"
                    >
                      Code
                    </div>
                    <ArrowDownIcon
                      class="h-3 w-3"
                      v-if="
                        params.field === 'category_code' &&
                        params.direction === 'desc'
                      "
                    />
                    <ArrowUpIcon
                      class="h-3 w-3"
                      v-if="
                        params.field === 'category_code' &&
                        params.direction === 'asc'
                      "
                    />
                  </div>
                </th>

                <th
                  scope="col"
                  class="
                    py-3
                    px-6
                    text-left text-xs
                    font-medium
                    text-gray-500
                    uppercase
                    tracking-wider
                  "
                >
                  <div class="flex justify-between">
                    <div
                      class="w-full"
                      style="cursor: pointer"
                      @click="$emit('sort', 'category_name')"
                    >
                      Category Name
                    </div>
                    <div>
                      <ArrowDownIcon
                        class="h-3 w-3"
                        v-if="
                          params.field === 'category_name' &&
                          params.direction === 'desc'
                        "
                      />
                    </div>

                    <ArrowUpIcon
                      class="h-3 w-3"
                      v-if="
                        params.field === 'category_name' &&
                        params.direction === 'asc'
                      "
                    />
                  </div>
                </th>

                <th
                  scope="col"
                  class="
                    px-6
                    py-3
                    text-left text-xs
                    font-medium
                    text-gray-500
                    uppercase
                    tracking-wider
                  "
                >
                  Action
                </th>
              </tr>
            </thead>
            <tbody v-if="values" class="bg-white divide-y divide-gray-200">
              <tr
                v-for="value in values.data"
                :key="value.id"
                class="hover:bg-blue-50 transition duration-350"
              >
                <td class="px-6 py-4 whitespace-nowrap">
                  <input
                    type="checkbox"
                    class="
                      apperance-none
                      checked:bg-blue-500
                      indeterminate:bg-gray-300
                      cursor-pointer
                      default:ring-2
                      hover:bg-blue-500
                      transition
                      duration-350
                      ease-out
                    "
                    :id="value.id"
                    v-model="checkbox.categories"
                    :value="value.id"
                  />
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex items-center">
                    <!-- <div class="flex-shrink-0 h-10 w-10">
                      <img
                        class="h-10 w-10 rounded-full"
                        src=""
                        alt="a"
                      />
                    </div> -->
                    <div>
                      <div class="text-sm font-medium text-gray-900">
                        {{ value.category_code }}
                      </div>
                    </div>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="text-sm text-gray-900">
                    {{ value.category_name }}
                  </div>
                </td>
                <td
                  class="
                    px-6
                    py-4
                    whitespace-nowrap
                    text-left text-sm
                    font-medium
                  "
                >
                  <!-- Edit -->
                  <div class="flex">
                    <a
                      href=""
                      @click.prevent="$emit('edit', value.id)"
                      class="text-indigo-600 hover:text-indigo-500"
                      title="Edit"
                    >
                      <div class="mx-1">
                        <PencilIcon class="h-5 w-5" />
                      </div>
                    </a>

                    <a
                      href=""
                      @click.prevent="$emit('destroy', value.id)"
                      class="text-indigo-600 hover:text-indigo-500"
                      title="Delete"
                    >
                      <div class="mx-1 text-center">
                        <TrashIcon class="h-5 w-5" />
                      </div>
                    </a>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { TrashIcon } from "@heroicons/vue/solid";
import { PencilIcon } from "@heroicons/vue/solid";
import { ArrowDownIcon } from "@heroicons/vue/solid";
import { ArrowUpIcon } from "@heroicons/vue/solid";

export default {
  props: {
    values: Object,
    params: Object,
    checkbox: Object,
  },
  components: {
    TrashIcon,
    PencilIcon,
    ArrowDownIcon,
    ArrowUpIcon,
  },
  data() {
    return {
      selectedAll: false,
    };
  },
  methods: {
    selectAll() {
      this.checkbox.categories = [];
      if (!this.checkbox.all) {
        for (let i in this.values.data) {
          this.checkbox.categories.push(this.values.data[i].id);
        }
      }
    },
  },
};
</script>