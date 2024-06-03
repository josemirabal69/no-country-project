<template>
  <NuxtLayout name="admin-layout">
    <div>
      <h2 class="my-5 text-3xl">Apoderados</h2>
      <Filters
        @on:filter="onFilter($event)"
        @on:remove="onRemoveFilters($event)"
      />
      <Table
        :columns="columns"
        :data="data"
        :loading="loading"
        @on:show-students="onShowStudents"
      />
      <Pagination
        v-model:page="page"
        :total="total"
        @update:page="onChangePage"
      />
    </div>
  </NuxtLayout>
</template>

<script setup lang="ts">
import Pagination from "@/components/tables/Pagination.vue";
import { columns, Table } from "@/components/tables/tutors";
import Filters from "@/components/tables/tutors/Filters.vue";
import { StudentTableDTO } from "@/dto/studentTableDTO";
import { TutorTableDTO } from "@/dto/tutorTableDTO";
import { useAdminStudentsStore } from "@/store/useAdminStudentsStore";
import { useAdminTutorsStore } from "@/store/useAdminTutorsStore";
import type { FilterApi } from "@/types/api/filters/filterApi";

const store = useAdminTutorsStore();
const studentsStore = useAdminStudentsStore();
const data: Ref<TutorTableDTO[]> = ref([]);
const total: Ref<number> = ref(0);
const limit: Ref<number> = ref(10);
const activeFilters: Ref<FilterApi[]> = ref([]);
const page: Ref<number> = ref(1);
const loading: Ref<boolean> = ref(true);

function onFilter(filters: FilterApi[]) {
  fetchStudents(0, limit.value, filters);
  activeFilters.value = filters;
  page.value = 1;
}

function onRemoveFilters(filters: FilterApi[]) {
  fetchStudents(0, limit.value, filters);
  activeFilters.value = filters;
  page.value = 1;
}

async function onShowStudents(tutor: TutorTableDTO) {
  tutor.loadingStudents = true;
  const { rows } = await studentsStore.getStudentsByTutor(tutor.id);
  tutor.students = StudentTableDTO.manyFromApiModel(rows);
  tutor.loadingStudents = false;
}

async function fetchStudents(
  offset: number,
  limit: number,
  filters: FilterApi[],
) {
  loading.value = true;
  const { rows, count } = await store.getTutors(offset, limit, filters);
  data.value = TutorTableDTO.manyFromApiModel(rows);
  total.value = count;
  loading.value = false;
}

async function onChangePage(page: number) {
  const offset = (page - 1) * limit.value;
  await fetchStudents(offset, limit.value, activeFilters.value);
}

onMounted(async () => {
  await fetchStudents(0, limit.value, activeFilters.value);
});
</script>