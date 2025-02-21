<template>
  <section class="editor-home">
    <header class="editor-header">
      <TopNav :project="project" />
    </header>
    <main v-if="userStore.userInfo" class="editor-main">
      <template v-if="projectName">
        <div v-if="isLoading" class="loading-wrapper">
          <NSpin size="large" />
        </div>
        <div v-else-if="error != null">
          {{ _t(error.userMessage) }}
        </div>
        <EditorContextProvider
          v-else-if="project != null"
          :project="project"
          :user-info="userStore.userInfo"
        >
          <ProjectEditor />
        </EditorContextProvider>
        <div v-else>TODO</div>
      </template>
      <template v-else>
        <ProjectList @selected="handleSelected" />
        <NButton @click="handleCreate">+</NButton>
      </template>
    </main>
  </section>
</template>

<script setup lang="ts">
import { watchEffect, computed, watch } from 'vue'
import { useRouter } from 'vue-router'
import { NButton, NSpin } from 'naive-ui'
import type { ProjectData } from '@/apis/project'
import { useUserStore } from '@/stores'
import { Project } from '@/models/project'
import TopNav from '@/components/top-nav/TopNav.vue'
import ProjectList from '@/components/project/ProjectList.vue'
import { useCreateProject } from '@/components/project'
import { getProjectEditorRoute } from '@/router'
import { useQuery } from '@/utils/exception'
import EditorContextProvider from './EditorContextProvider.vue'
import ProjectEditor from './ProjectEditor.vue'

const localCacheKey = 'TODO_GOPLUS_BUILDER_CACHED_PROJECT'

const userStore = useUserStore()
watchEffect(() => {
  // This will be called on mount and whenever userStore changes,
  // which are the cases when userStore.signOut() is called
  if (!userStore.hasSignedIn()) {
    userStore.signInWithRedirection()
  }
})

const router = useRouter()
const createProject = useCreateProject()
const projectName = computed(
  () => router.currentRoute.value.params.projectName as string | undefined
)

const {
  data: project,
  isFetching: isLoading,
  error
} = useQuery(
  async () => {
    if (userStore.userInfo == null) return null
    if (projectName.value == null) return null
    // TODO: UI logic to handle conflicts when there are local cache
    const newProject = new Project()
    await newProject.loadFromCloud(userStore.userInfo.name, projectName.value)
    newProject.syncToLocalCache(localCacheKey)
    return newProject
  },
  { en: 'Load project failed', zh: '加载项目失败' }
)

watch(
  // https://vuejs.org/guide/essentials/watchers.html#deep-watchers
  // According to the document, we should use `() => project.value` instead of
  // `project` to avoid deep watching, which is not expected here.
  () => project.value,
  (_, oldProject) => {
    oldProject?.dispose()
    ;(window as any).project = project.value // for debug purpose, TODO: remove me
  }
)

function openProject(projectName: string) {
  router.push(getProjectEditorRoute(projectName))
}

function handleSelected(project: ProjectData) {
  openProject(project.name)
}

async function handleCreate() {
  const newProject = await createProject()
  openProject(newProject.name)
}
</script>

<style scoped lang="scss">
.editor-home {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;

  background-color: #e9fcff; // TODO: define as UI vars
}

.editor-header {
  flex: 0 0 auto;
}

.editor-main {
  flex: 1 1 0;
  display: flex;
}

.loading-wrapper {
  width: 100%;
  height: 400px;
  display: flex;
  justify-content: center;
}
</style>
