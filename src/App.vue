<script setup lang="ts">
import { reactive, ref, watch } from 'vue'
interface Project {
  id: number
  name: string
  goal: string
  status: '进行中' | '已完成'
}
interface Issue {
  id: number
  projectId: number
  title: string
  description: string
  status: '待解决' | '已解决'
}
interface Practice {
  id: number
  issueId: number
  title: string
  content: string
  progress: '准备尝试' | '尝试中' | '已尝试'
  result: '未评估' | '有效' | '无效' | '不适用'
}
const projectNameMaxLength = 50
const appName: string = 'DevTrail'
const description: string = '面向开发者的项目学习与资料工作台'

const defaultProjects: Project[] = [
  {
    id: 1,
    name: '开发 DevTrail',
    goal: '学习 Vue3 和 TypeScript，完成第一个完整项目',
    status: '进行中',
  },
  {
    id: 2,
    name: '学习 TypeScript',
    goal: '掌握接口、联合类型和函数参数类型',
    status: '进行中',
  },
  {
    id: 3,
    name: '学习 Vue3',
    goal: '掌握 Vue3 的基础语法和组件化开发',
    status: '进行中',
  },
  {
    id: 4,
    name: '测试',
    goal: '掌握 Vue3 的基础语法和组件化开发',
    status: '进行中',
  },
]

const storageError = ref('')
let initialProjects: Project[] = defaultProjects



try {
  const savedProjects = localStorage.getItem('devtrail-projects')

  if (savedProjects !== null) {
    const parsedProjects: unknown = JSON.parse(savedProjects)

  if (
    Array.isArray(parsedProjects) &&
    parsedProjects.every((item) =>
      item !== null &&
      typeof item === 'object' &&
      typeof item.id === 'number' &&
      Number.isSafeInteger(item.id) &&
      item.id > 0 &&
      typeof item.name === 'string' &&
      item.name.trim().length > 0 &&
      typeof item.goal === 'string' &&
      item.goal.trim().length > 0 &&
      (item.status === '进行中' || item.status === '已完成')
    ) &&
    new Set(parsedProjects.map((item) => item.id)).size === parsedProjects.length
  ) {
    initialProjects = parsedProjects.map((item): Project => ({ id: item.id, name: item.name, goal: item.goal, status: item.status === '已完成' ? '已完成' : '进行中' }))
  } else {
    storageError.value = '保存的项目数据格式不正确，暂时显示示例项目。'
  }
    }
  } catch {
    storageError.value = '读取项目失败，暂时显示示例项目。'
  }

const projects = reactive<Project[]>(initialProjects)

const defaultIssues: Issue[] = projects.length > 0 ? [
  {
    id: 101,
    projectId: projects[0]?.id ?? 1,
    title: '如何在项目下展示对应的问题？',
    description: '每个项目只显示属于自己的问题，避免不同项目的问题混在一起。',
    status: '待解决',
  },
] : []

const issueStorageError = ref('')
let initialIssues: Issue[] = defaultIssues
let issueOwnershipReliable = !storageError.value

try {
  const savedIssues = localStorage.getItem('devtrail-issues')

  if (savedIssues !== null) {
    const parsedIssues: unknown = JSON.parse(savedIssues)

    if (
      Array.isArray(parsedIssues) &&
      parsedIssues.every((item) =>
        item !== null &&
        typeof item === 'object' &&
        Number.isSafeInteger(item.id) &&
        item.id > 0 &&
        Number.isSafeInteger(item.projectId) &&
        item.projectId > 0 &&
        typeof item.title === 'string' &&
        item.title.trim().length > 0 &&
        typeof item.description === 'string' &&
        item.description.trim().length > 0 &&
        (item.status === '待解决' || item.status === '已解决')
      ) &&
      new Set(parsedIssues.map((item) => item.id)).size === parsedIssues.length
    ) {
      // 项目读取失败时，无法可靠判断归属，保留原问题数据。
      initialIssues = storageError.value
        ? parsedIssues
        : parsedIssues.filter((issue) =>
            projects.some((project) => project.id === issue.projectId)
          )

      if (initialIssues.length !== parsedIssues.length) {
        issueStorageError.value = '已从当前列表移除所属项目不存在的问题；下次成功保存问题时更新本地存储。'
      }
    } else {
      issueStorageError.value = '保存的问题数据格式不正确，暂时显示示例问题。'
      issueOwnershipReliable = false
    }
  }
} catch {
  issueStorageError.value = '读取问题失败，暂时显示示例问题。'
  issueOwnershipReliable = false
}

const issues = reactive<Issue[]>(initialIssues)
const practiceStorageError = ref('')
let initialPractices: Practice[] = []

try {
  const savedPractices = localStorage.getItem('devtrail-practices')

  if (savedPractices !== null) {
    const parsedPractices: unknown = JSON.parse(savedPractices)

    if (
      Array.isArray(parsedPractices) &&
      parsedPractices.every((item) =>
        item !== null &&
        typeof item === 'object' &&
        Number.isSafeInteger(item.id) &&
        item.id > 0 &&
        Number.isSafeInteger(item.issueId) &&
        item.issueId > 0 &&
        typeof item.title === 'string' &&
        item.title.trim().length > 0 &&
        typeof item.content === 'string' &&
        (item.progress === '准备尝试' || item.progress === '尝试中' || item.progress === '已尝试') &&
        (item.result === '未评估' || item.result === '有效' || item.result === '无效' || item.result === '不适用')
      ) &&
      new Set(parsedPractices.map((item) => item.id)).size === parsedPractices.length
    ) {
      // 上游读取异常时，无法可靠判断归属，保留原实践数据。
      initialPractices = issueOwnershipReliable
        ? parsedPractices.filter((practice) =>
            issues.some((issue) => issue.id === practice.issueId)
          )
        : parsedPractices

      if (initialPractices.length !== parsedPractices.length) {
        practiceStorageError.value = '已从当前列表移除所属问题不存在的实践；下次成功保存实践时更新本地存储。'
      }
    } else {
      practiceStorageError.value = '保存的实践数据格式不正确，暂时显示空列表。'
    }
  }
} catch {
  practiceStorageError.value = '读取实践失败，暂时显示空列表。'
}

const practices = reactive<Practice[]>(initialPractices)
const addingPracticeIssueId = ref<number | null>(null)
const newPracticeTitle = ref('')
const newPracticeContent = ref('')
const practiceFormError = ref('')
const editingPracticeId = ref<number | null>(null)
const editingPracticeTitle = ref('')
const editingPracticeContent = ref('')
const editingPracticeProgress = ref<Practice['progress']>('准备尝试')
const editingPracticeResult = ref<Practice['result']>('未评估')
const practiceEditError = ref('')
const pageMinimumHeight = ref(0)

function preservePageHeight(element: Element) {
  pageMinimumHeight.value = Math.ceil(window.scrollY + window.innerHeight) + 2
  const focusedElement = document.activeElement
  if (focusedElement instanceof HTMLElement && element.contains(focusedElement)) {
    focusedElement.blur()
  }
}

function finishFormExpansion() {
  // 等表单撑开再释放旧占位，同时避免页面底部缩短迫使浏览器回滚。
  pageMinimumHeight.value = Math.ceil(window.scrollY + window.innerHeight) + 2
}

function cancelEditPractice() {
  editingPracticeId.value = null
  editingPracticeTitle.value = ''
  editingPracticeContent.value = ''
  editingPracticeProgress.value = '准备尝试'
  editingPracticeResult.value = '未评估'
  practiceEditError.value = ''
}

function startEditPractice(practice: Practice) {
  cancelEditPractice()
  editingPracticeId.value = practice.id
  editingPracticeTitle.value = practice.title
  editingPracticeContent.value = practice.content
  editingPracticeProgress.value = practice.progress
  editingPracticeResult.value = practice.result
}

function saveEditPractice(practice: Practice) {
  const title = editingPracticeTitle.value.trim()
  const content = editingPracticeContent.value.trim()

  if (!title) {
    practiceEditError.value = '请填写实践标题'
    return
  }

  practice.title = title
  practice.content = content
  practice.progress = editingPracticeProgress.value
  practice.result = editingPracticeResult.value
  cancelEditPractice()
}

function deletePractice(practice: Practice) {
  const confirmed = window.confirm(`确定删除实践“${practice.title}”吗？`)

  if (!confirmed) {
    return
  }

  if (editingPracticeId.value === practice.id) {
    cancelEditPractice()
  }

  const index = practices.findIndex((item) => item.id === practice.id)

  if (index !== -1) {
    practices.splice(index, 1)
  }
}

function cancelAddPractice() {
  addingPracticeIssueId.value = null
  newPracticeTitle.value = ''
  newPracticeContent.value = ''
  practiceFormError.value = ''
}

function startAddPractice(issue: Issue) {
  cancelAddPractice()
  addingPracticeIssueId.value = issue.id
}

function addPractice(issue: Issue) {
  const title = newPracticeTitle.value.trim()
  const content = newPracticeContent.value.trim()

  if (!title) {
    practiceFormError.value = '请填写实践标题'
    return
  }

  let newId = Date.now()
  while (practices.some((item) => item.id === newId)) {
    newId++
  }

  practices.push({
    id: newId,
    issueId: issue.id,
    title,
    content,
    progress: '准备尝试',
    result: '未评估',
  })
  cancelAddPractice()
}
watch(projects, () => {
  try {
    localStorage.setItem('devtrail-projects', JSON.stringify(projects))
    storageError.value = ''
  } catch {
    storageError.value = '保存项目失败，当前修改可能在刷新后丢失。'
  }
})

watch(issues, () => {
  try {
    localStorage.setItem('devtrail-issues', JSON.stringify(issues))
    issueStorageError.value = ''
  } catch {
    issueStorageError.value = '保存问题失败，当前修改可能在刷新后丢失。'
  }
})

watch(practices, () => {
  try {
    localStorage.setItem('devtrail-practices', JSON.stringify(practices))
    practiceStorageError.value = ''
  } catch {
    practiceStorageError.value = '保存实践失败，当前修改可能在刷新后丢失。'
  }
})

const newProjectName = ref('')
const newProjectGoal = ref('')
const formError = ref('')
const editingProjectId = ref<number | null>(null)
const editingProjectName = ref('')
const editingProjectGoal = ref('')
const editError = ref('')
const addingIssueProjectId = ref<number | null>(null)
const newIssueTitle = ref('')
const newIssueDescription = ref('')
const issueFormError = ref('')
const editingIssueId = ref<number | null>(null)
const editingIssueTitle = ref('')
const editingIssueDescription = ref('')
const issueEditError = ref('')

function cancelAddIssue() {
  addingIssueProjectId.value = null
  newIssueTitle.value = ''
  newIssueDescription.value = ''
  issueFormError.value = ''
}

function startAddIssue(project: Project) {
  cancelAddIssue()
  addingIssueProjectId.value = project.id
}

function addIssue(project: Project) {
  const title = newIssueTitle.value.trim()
  const description = newIssueDescription.value.trim()

  if (!title || !description) {
    issueFormError.value = '请填写问题标题和描述'
    return
  }

  let newId = Date.now()

  while (issues.some((item) => item.id === newId)) {
    newId++
  }

  issues.push({
    id: newId,
    projectId: project.id,
    title,
    description,
    status: '待解决',
  })

  cancelAddIssue()
}

function toggleIssueStatus(issue: Issue) {
  if (issue.status === '待解决') {
    issue.status = '已解决'
  } else {
    issue.status = '待解决'
  }
}

function startEditIssue(issue: Issue) {
  issueEditError.value = ''
  editingIssueTitle.value = issue.title
  editingIssueDescription.value = issue.description
  editingIssueId.value = issue.id
}

function cancelEditIssue() {
  issueEditError.value = ''
  editingIssueId.value = null
  editingIssueTitle.value = ''
  editingIssueDescription.value = ''
}

function saveEditIssue(issue: Issue) {
  const title = editingIssueTitle.value.trim()
  const description = editingIssueDescription.value.trim()

  if (!title || !description) {
    issueEditError.value = '请填写问题标题和描述'
    return
  }

  issue.title = title
  issue.description = description
  cancelEditIssue()
}

function deleteIssue(issue: Issue) {
  const confirmed = window.confirm(`确定删除“${issue.title}”吗？`)

  if (!confirmed) {
    return
  }

  const index = issues.findIndex((item) => item.id === issue.id)

  if (index !== -1) {
    if (addingPracticeIssueId.value === issue.id) {
      cancelAddPractice()
    }

    if (practices.some((item) =>
      item.id === editingPracticeId.value && item.issueId === issue.id
    )) {
      cancelEditPractice()
    }

    for (let i = practices.length - 1; i >= 0; i--) {
      if (practices[i]?.issueId === issue.id) {
        practices.splice(i, 1)
      }
    }

    issues.splice(index, 1)

    if (editingIssueId.value === issue.id) {
      cancelEditIssue()
    }
  }
}

function startEdit(project: Project) {
  editingProjectName.value = project.name
  editingProjectGoal.value = project.goal
  editingProjectId.value = project.id
  editError.value = ''
}

function cancelEdit() {
  editingProjectId.value = null
  editingProjectName.value = ''
  editingProjectGoal.value = ''
  editError.value = ''
}

function saveEdit(project: Project) {
  const name = editingProjectName.value.trim()
  const goal = editingProjectGoal.value.trim()

  if (!name || !goal) {
    editError.value = '请填写项目名称和项目目标'
    return
  }

  if (name.length > projectNameMaxLength) {
    editError.value = '项目名称不能超过 50 个字符'
    return
  }

  project.name = name
  project.goal = goal
  cancelEdit()
}

function addProject() {
  const name = newProjectName.value.trim()
  const goal = newProjectGoal.value.trim()

   if (!name || !goal) {
    formError.value = '请填写项目名称和项目目标'
    return
  }
  if (name.length > projectNameMaxLength) {
    formError.value = '项目名称不能超过 50 个字符'
    return
  }
  formError.value = ''

  let newId = Date.now()

  while (projects.some((item) => item.id === newId)) {
    newId++
  }

  projects.push({
    id: newId,
    name,
    goal,
    status: '进行中',
  })
  newProjectName.value = ''
  newProjectGoal.value = ''
}

function toggleProjectStatus(project: Project) {
  if (project.status === '进行中') {
    project.status = '已完成'
  } else {
    project.status = '进行中'
  }
}

function deleteProject(project: Project) {
  const confirmed = window.confirm(`确定删除“${project.name}”吗？`)
  if(!confirmed){
    return
}
  const index = projects.findIndex((item) => item.id === project.id)
  if (index !== -1) {
    if (addingIssueProjectId.value === project.id) {
      cancelAddIssue()
    }
    
    if (issues.some((item) =>
      item.projectId === project.id &&
      item.id === editingIssueId.value
    )) {
      cancelEditIssue()
    }
    
    const projectIssueIds = new Set(
      issues.filter((item) => item.projectId === project.id).map((item) => item.id)
    )

    if (
      addingPracticeIssueId.value !== null &&
      projectIssueIds.has(addingPracticeIssueId.value)
    ) {
      cancelAddPractice()
    }

    if (practices.some((item) =>
      item.id === editingPracticeId.value && projectIssueIds.has(item.issueId)
    )) {
      cancelEditPractice()
    }

    for (let i = practices.length - 1; i >= 0; i--) {
      const practice = practices[i]
      if (practice && projectIssueIds.has(practice.issueId)) {
        practices.splice(i, 1)
      }
    }

    for (let i = issues.length - 1; i >= 0; i--) {
      if (issues[i]?.projectId === project.id) {
        issues.splice(i, 1)
      }
    }

    projects.splice(index, 1)
    if (editingProjectId.value === project.id) {
      cancelEdit()
    }
  }
}
</script>

<template>
  <main class="home" :style="{ minHeight: pageMinimumHeight ? `${pageMinimumHeight}px` : undefined }">
    <header class="page-header">
      <h1>{{ appName }}</h1>
      <p>{{ description }}</p>
    </header>

    <section>
      <h2>我的学习项目</h2>

      <Transition name="surface">
        <p v-if="storageError" class="form-error" role="alert">
        {{ storageError }}
      </p>
      </Transition>

      <Transition name="surface">
        <p v-if="issueStorageError" class="form-error" role="alert">
        {{ issueStorageError }}
      </p>
      </Transition>  

      <Transition name="surface">
        <p v-if="practiceStorageError" class="form-error" role="alert">
          {{ practiceStorageError }}
        </p>
      </Transition>

      <form class="project-form" @submit.prevent="addProject">
        <div>
          <label for="project-name">项目名称</label>
          <input
            id="project-name"
            v-model="newProjectName" :maxlength="projectNameMaxLength" aria-describedby="project-name-hint"
            type="text"
            placeholder="例如：学习 Vue3"
          />
          <small id="project-name-hint" class="name-hint">{{ newProjectName.length }}/{{ projectNameMaxLength }}，项目名称最多 50 个字符</small>
        </div>

        <div>
          <label for="project-goal">项目目标</label>
          <textarea id="project-goal" v-model="newProjectGoal" rows="3" placeholder="例如：掌握组件之间的数据传递"></textarea>
        </div>
        <Transition name="surface">
          <p v-if="formError" class="form-error" role="alert">
          {{ formError }}
        </p>
        </Transition>
        <button type="submit" class="status-button">
          新增项目
        </button>
      </form>

      <Transition name="surface">
        <p v-if="projects.length === 0">
        还没有学习项目，请通过上方表单新增项目。
      </p>
      </Transition>

      <TransitionGroup name="surface">
      <article 
      v-for="project in projects"
      :key="project.id"
      class="project-card"
      >
        <div class="project-overview">
        <div class="project-meta"><span class="project-label">学习项目</span><span class="status">{{ project.status }}</span></div>
        <h3>{{ project.name }}</h3>
        <p>{{ project.goal }}</p>
        </div>
        
      <section class="project-issues">
        <div class="issues-toolbar">
        <h4>项目问题 <span class="issue-count">{{ issues.filter((item) => item.projectId === project.id).length }}</span></h4>

        <button
          type="button"
          class="status-button issue-add-button"
          @click="startAddIssue(project)"
        >
          新增问题
        </button>
        </div>

        <Transition name="practice-collapse" @after-enter="finishFormExpansion" @before-leave="preservePageHeight">
          <div v-if="addingIssueProjectId === project.id" class="practice-collapse"><div class="practice-collapse-inner"><form
          class="project-form issue-form"
          @submit.prevent="addIssue(project)"
        >
          <div>
          <label :for="`issue-title-${project.id}`">问题标题</label>
          <input
            :id="`issue-title-${project.id}`"
            v-model="newIssueTitle"
            type="text"
          />
          </div>

          <div>
          <label :for="`issue-description-${project.id}`">问题描述</label>
          <textarea
            :id="`issue-description-${project.id}`"
            v-model="newIssueDescription"
            rows="3"
          ></textarea>
          </div>

          <Transition name="surface">
            <p v-if="issueFormError" class="form-error" role="alert">
            {{ issueFormError }}
          </p>
          </Transition>

          <div class="edit-actions">
          <button type="submit" class="status-button save-button">保存问题</button>


          <button
            type="button"
            class="status-button cancel-button"
            @click="cancelAddIssue"
          >
            取消
          </button> 
          </div>
        </form></div></div>
        </Transition>

        <TransitionGroup name="surface" tag="ul" class="issue-list">
          <li
            v-for="issue in issues.filter(
              (item) => item.projectId === project.id
            )"
            :key="issue.id"
            class="issue-card"
          >
            <div class="issue-heading">
              <strong class="issue-title">{{ issue.title }}</strong>
              <span class="issue-status" :class="{ 'is-resolved': issue.status === '已解决' }">
                {{ issue.status }}
              </span>
            </div>
            <p class="issue-description">{{ issue.description }}</p>
            <div class="practice-toolbar">
              <span>实践记录：{{ practices.filter((item) => item.issueId === issue.id).length }} 条</span>
              <button
                type="button"
                class="status-button practice-add-button"
                @click="startAddPractice(issue)"
              >
                新增实践
              </button>
            </div>
            <Transition name="practice-collapse" @after-enter="finishFormExpansion" @before-leave="preservePageHeight">
              <div v-if="addingPracticeIssueId === issue.id" class="practice-collapse"><div class="practice-collapse-inner"><form
                class="project-form issue-form practice-form"
                @submit.prevent="addPractice(issue)"
              >
                <div>
                  <label :for="`practice-title-${issue.id}`">实践标题</label>
                  <input
                    :id="`practice-title-${issue.id}`"
                    v-model="newPracticeTitle"
                    type="text"
                  />
                </div>
                <div>
                  <label :for="`practice-content-${issue.id}`">尝试过程（可选）</label>
                  <textarea
                    :id="`practice-content-${issue.id}`"
                    v-model="newPracticeContent"
                    rows="3"
                  ></textarea>
                </div>
                <Transition name="surface">
                  <p v-if="practiceFormError" class="form-error" role="alert">
                    {{ practiceFormError }}
                  </p>
                </Transition>
                <div class="edit-actions">
                  <button type="submit" class="status-button save-button">
                    保存实践
                  </button>
                  <button
                    type="button"
                    class="status-button cancel-button"
                    @click="cancelAddPractice"
                  >
                    取消
                  </button>
                </div>
              </form></div></div>
            </Transition>
            <ul class="practice-list">
              <li
                v-for="practice in practices.filter(
                  (item) => item.issueId === issue.id
                )"
                :key="practice.id"
              >
                <strong>{{ practice.title }}</strong>
                <p class="issue-description">
                  {{ practice.content || '尚未记录尝试过程' }}
                </p>
                <p class="practice-meta">进度：{{ practice.progress }} · 结果：{{ practice.result }}</p>
                <button
                  type="button"
                  class="status-button practice-add-button practice-edit-button"
                  @click="startEditPractice(practice)"
                >
                  编辑实践
                </button>
                <button
                  type="button"
                  class="status-button practice-add-button practice-edit-button"
                  @click="deletePractice(practice)"
                >
                  删除实践
                </button>
                <Transition name="practice-collapse" @after-enter="finishFormExpansion" @before-leave="preservePageHeight">
                  <div v-if="editingPracticeId === practice.id" class="practice-collapse">
                    <div class="practice-collapse-inner">
                  <form
                    class="project-form issue-form practice-form"
                    @submit.prevent="saveEditPractice(practice)"
                  >
                    <div>
                      <label :for="`edit-practice-title-${practice.id}`">实践标题</label>
                      <input
                        :id="`edit-practice-title-${practice.id}`"
                        v-model="editingPracticeTitle"
                        type="text"
                      />
                    </div>
                    <div>
                      <label :for="`edit-practice-content-${practice.id}`">尝试过程（可选）</label>
                      <textarea
                        :id="`edit-practice-content-${practice.id}`"
                        v-model="editingPracticeContent"
                        rows="3"
                      ></textarea>
                    </div>
                    <div>
                      <label :for="`edit-practice-progress-${practice.id}`">实践进度</label>
                      <select
                        :id="`edit-practice-progress-${practice.id}`"
                        v-model="editingPracticeProgress"
                      >
                        <option value="准备尝试">准备尝试</option>
                        <option value="尝试中">尝试中</option>
                        <option value="已尝试">已尝试</option>
                      </select>
                    </div>
                    <div>
                      <label :for="`edit-practice-result-${practice.id}`">实践结果</label>
                      <select
                        :id="`edit-practice-result-${practice.id}`"
                        v-model="editingPracticeResult"
                      >
                        <option value="未评估">未评估</option>
                        <option value="有效">有效</option>
                        <option value="无效">无效</option>
                        <option value="不适用">不适用</option>
                      </select>
                    </div>
                    <Transition name="surface">
                      <p v-if="practiceEditError" class="form-error" role="alert">
                        {{ practiceEditError }}
                      </p>
                    </Transition>
                    <div class="edit-actions">
                      <button type="submit" class="status-button save-button">保存修改</button>
                      <button type="button" class="status-button cancel-button" @click="cancelEditPractice">
                        取消
                      </button>
                    </div>
                  </form>
                    </div>
                  </div>
                </Transition>
              </li>
            </ul>


            <div class="issue-actions">

              <button
                type="button"
                class="status-button issue-action-button"
                :class="{ 'is-resolved': issue.status === '已解决' }"
                @click="toggleIssueStatus(issue)"
              >
                {{ issue.status === '待解决' ? '标记为已解决' : '恢复待解决' }}
              </button>
              <button
                type="button"
                class="status-button issue-action-button"
                :class="{ 'is-resolved': issue.status === '已解决' }"
                @click="startEditIssue(issue)"
              >
                编辑问题
              </button>
              <button
                type="button"
                class="status-button issue-action-button"
                :class="{ 'is-resolved': issue.status === '已解决' }"
                @click="deleteIssue(issue)"
              >
                删除问题
              </button>
            </div>
            <Transition name="practice-collapse" @after-enter="finishFormExpansion" @before-leave="preservePageHeight">
              <div v-if="editingIssueId === issue.id" class="practice-collapse"><div class="practice-collapse-inner"><form
              class="project-form issue-form"
              @submit.prevent="saveEditIssue(issue)"
            >
              <div>
                <label :for="`edit-issue-title-${issue.id}`">问题标题</label>
                <input
                  :id="`edit-issue-title-${issue.id}`"
                  v-model="editingIssueTitle"
                  type="text"
                />
              </div>

              <div>
                <label :for="`edit-issue-description-${issue.id}`">问题描述</label>
                <textarea
                  :id="`edit-issue-description-${issue.id}`"
                  v-model="editingIssueDescription"
                  rows="3"
                ></textarea>
              </div>
              <Transition name="surface">
                <p v-if="issueEditError" class="form-error" role="alert">
                {{ issueEditError }}
              </p>
              </Transition>
              <div class="edit-actions">
                <button type="submit" class="status-button save-button">保存修改</button>
              <button
                type="button"
                class="status-button cancel-button"
                @click="cancelEditIssue"
              >
                取消
              </button>
              </div>
            </form></div></div>
            </Transition>
          </li>
        </TransitionGroup>
      </section>

        <Transition name="practice-collapse" @after-enter="finishFormExpansion" @before-leave="preservePageHeight">
          <div v-if="editingProjectId === project.id" class="practice-collapse"><div class="practice-collapse-inner"><form
          class="project-form"
          @submit.prevent="saveEdit(project)"
        >
          <div>
            <label :for="`edit-name-${project.id}`">项目名称</label>
            <input
              :id="`edit-name-${project.id}`"
              v-model="editingProjectName" :maxlength="projectNameMaxLength" :aria-describedby="`edit-name-hint-${project.id}`"
              type="text"
            />
            <small :id="`edit-name-hint-${project.id}`" class="name-hint">{{ editingProjectName.length }}/{{ projectNameMaxLength }}，项目名称最多 50 个字符</small>
          </div>
        
          <div>
            <label :for="`edit-goal-${project.id}`">项目目标</label>
            <textarea :id="`edit-goal-${project.id}`" v-model="editingProjectGoal" rows="3"></textarea>
          </div>
          <Transition name="surface">
            <p v-if="editError" class="form-error" role="alert">
            {{ editError }}
          </p>
          </Transition>
          <div class="edit-actions">
            <button type="submit" class="status-button save-button">
              保存
            </button>
            <button type="button" class="status-button cancel-button" @click="cancelEdit">
              取消
            </button>
          </div>
        </form></div></div>
        </Transition>
        <footer class="project-footer">
        <span class="project-footer-label">项目操作</span>
        <div class="project-actions">
          <button type="button" class="status-button" @click="toggleProjectStatus(project)">
            {{ project.status === '进行中' ? '标记为已完成' : '恢复进行中' }}
          </button> 
          <button type="button" class="status-button" @click="startEdit(project)">
            {{ editingProjectId === project.id ? '编辑中' : '编辑项目' }}
          </button>
          <button type="button" class="status-button" @click="deleteProject(project)">
            删除项目
          </button>
        </div>
        </footer>
      </article>
      </TransitionGroup>
    </section>
  </main>
</template>

<style scoped>

@property --button-start {
  syntax: '<color>';
  inherits: false;
  initial-value: #ddf4f5;
}

@property --button-end {
  syntax: '<color>';
  inherits: false;
  initial-value: #dbf1e6;
}

.home {
  max-width: 960px;
  margin: 0 auto;
  padding: 48px 24px;
}

.page-header {
  margin-bottom: 40px;
}

h1 {
  margin: 0;
  color: #15803d;
  font-size: 36px;
}

p {
  color: #64748b;
  line-height: 1.7;
}

.project-card {
  padding: 28px;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  background: white;
  box-shadow: 0 4px 20px rgb(15 23 42 / 5%);
  transition: box-shadow 0.2s, border-color 0.2s;
}

.project-card + .project-card {
  margin-top: 16px;
}

.project-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 8px 28px rgb(15 23 42 / 9%);
}

.project-card h3,
.project-card p {
  overflow-wrap: anywhere;
}

.project-card h3 {
  margin: 18px 0 8px;
  color: #0f172a;
  font-size: 20px;
}

.project-card p {
  margin: 0 0 24px;
  font-size: 14px;
}

/* .status {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 6px;
  background: #dcfce7;
  color: #166534;
  font-size: 14px;
} */

.status {
  display: inline-block;
  padding: 4px 10px;
  border: 1px solid #dbeafe;
  border-radius: 20px;
  background: #eff6ff;
  color: #1d4ed8;
  font-size: 12px;
  font-weight: 500;
}

.status-button {
  padding: 8px 14px;
  border: 1px solid #a7dfba;
  border-radius: 8px;
  --button-start: #ddf4f5;
  --button-end: #dbf1e6;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  color: #166534;
  font: inherit;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 2px 4px rgb(22 101 52 / 6%);
  transition: --button-start 0.65s ease, --button-end 0.65s ease,
    color 0.65s ease, border-color 0.65s ease,
    box-shadow 0.65s ease, transform 0.2s ease;
}

.project-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.status-button:hover {
  --button-start: #e4f3f4;
  --button-end: #dceee5;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  box-shadow: 0 4px 8px rgb(22 101 52 / 12%);
  transform: translateY(-1px);
}

.status-button:active {
  transform: translateY(0);
  box-shadow: none;
}

.status-button:focus-visible {
  outline: 3px solid #86efac;
  outline-offset: 3px;
}

.form-error {
  color: #b91c1c;
  font-size: 14px;
}

.project-form {
  display: grid;
  gap: 16px;
  margin-bottom: 24px;
  padding: 24px;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  background: white;
}

.project-form > div {
  display: grid;
  gap: 8px;
}

.project-form label {
  color: #334155;
  font-size: 14px;
  font-weight: 500;
}

.project-form input,
.project-form textarea,
.project-form select {
  box-sizing: border-box;
  width: 100%;
  min-width: 0;
  padding: 10px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font: inherit;
  color: #0f172a;
  background: white;
}

.project-form textarea {
  resize: vertical;
}

.project-form input:focus-visible,
.project-form textarea:focus-visible,
.project-form select:focus-visible {
  outline: 2px solid #15803d;
  outline-offset: 2px;
}

.project-form .status-button {
  justify-self: start;
}

.project-form .form-error {
  margin: 0;
}

.project-form > .edit-actions {
  display: flex;
  gap: 12px;
}

.edit-actions > .status-button + .status-button {
  margin-left: 0;
}

.save-button,
.cancel-button {
  border-color: #ddd2ed;
  --button-start: #f5f0fb;
  --button-end: #ece3f7;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  color: #685080;
  box-shadow: 0 2px 4px rgb(104 80 128 / 6%);
}

.save-button:hover,
.cancel-button:hover {
  --button-start: #eee5f8;
  --button-end: #e5d8f2;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  box-shadow: 0 4px 8px rgb(104 80 128 / 10%);
}

.save-button:active,
.cancel-button:active {
  box-shadow: none;
}

.save-button:focus-visible,
.cancel-button:focus-visible {
  outline-color: #b69bce;
}
.name-hint {
  color: #94a3b8;
  font-size: 12px;
}

.project-issues {
  margin: 24px 0;
  padding: 20px;
  border: 1px solid #e5eae9;
  border-radius: 14px;
  background: #f4f7f6;
}

.issue-form {
  margin: 0 0 20px;
  padding: 20px;
  border-radius: 10px;
  border-left: 3px solid #9bb8ac;
}

.issue-add-button {
  margin: 0;
}

.project-issues h4 {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0;
  color: #475569;
  font-size: 13px;
  font-weight: 600;
}

.project-issues h4::before {
  content: '';
  width: 4px;
  height: 14px;
  border-radius: 4px;
  background: #7c9e94;
}

.issue-list {
  display: grid;
  gap: 10px;
  margin: 0;
  padding: 0;
  list-style: none;
}

.practice-toolbar {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px 12px;
  margin-top: 18px;
  color: #78818f;
  font-size: 12px;
}

.practice-add-button {
  padding: 3px 9px;
  border-color: #ded7ec;
  border-radius: 6px;
  --button-start: #f4f1fa;
  --button-end: #ece7f5;
  color: #75618e;
  font-size: 12px;
  font-weight: 500;
  line-height: 1.5;
  box-shadow: none;
}

.practice-add-button:hover {
  --button-start: #eee8f7;
  --button-end: #e5dcf1;
  box-shadow: none;
}

.practice-add-button:focus-visible {
  outline-color: #b69bce;
}

.practice-list {
  display: grid;
  gap: 10px;
  margin: 14px 0 0;
  padding: 0;
  list-style: none;
}

.practice-list:empty {
  display: none;
}

.practice-list > li {
  min-width: 0;
  padding: 12px 14px;
  border-left: 3px solid #d8cde6;
  border-radius: 4px 8px 8px 4px;
  background: #f8f7fb;
  color: #655875;
  font-size: 13px;
  overflow-wrap: anywhere;
}

.project-card .practice-meta {
  margin: 10px 0 0;
  color: #8a8096;
  font-size: 12px;
}

.practice-form {
  margin-top: 20px;
  border-left-color: #c6b5d9;
}

.practice-edit-button {
  margin-top: 12px;
}

.practice-edit-button + .practice-edit-button {
  margin-left: 10px;
}

.issue-card {
  min-width: 0;
  padding: 16px 18px;
  border: 1px solid #e2eae7;
  border-radius: 8px;
  background: #fff;
}

.issue-heading {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px 12px;
}


.issue-actions {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid #edf0ee;
}

.issue-actions + .issue-form,
.issue-actions + .practice-collapse .issue-form {
  margin-top: 20px;
}


.issue-actions .issue-action-button {
  padding: 6px 11px;
  font-size: 12px;
  line-height: 1.5;
}

@media (max-width: 480px) {
  .issue-actions {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .issue-actions .issue-action-button:first-child {
    grid-column: 1 / -1;
  }

  .issue-actions .issue-action-button {
    min-width: 0;
    white-space: normal;
  }
}

.issue-title {
  flex: 1 1 180px;
  min-width: 0;
  color: #263e39;
  font-size: 14px;
  font-weight: 600;
  line-height: 1.6;
  overflow-wrap: anywhere;
}

.issue-status {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 3px 9px;
  border: 1px solid #eadcb9;
  border-radius: 999px;
  background: #fff8e8;
  color: #805d19;
  font-size: 12px;
  line-height: 1.5;
  white-space: nowrap;
  transition: background-color 0.65s ease, color 0.65s ease, border-color 0.65s ease;
}

.issue-status::before {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: currentColor;
}

.issue-status.is-resolved {
  border-color: #bfdfcd;
  background: #edf8f1;
  color: #277047;
}

.issue-action-button {
  flex-shrink: 0;
  border-color: #eadcb9;
  --button-start: #fffaf0;
  --button-end: #fff3d9;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  color: #805d19;
  box-shadow: 0 2px 4px rgb(128 93 25 / 6%);
}

.issue-action-button:hover {
  --button-start: #fbf2df;
  --button-end: #f5e7c6;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  box-shadow: 0 4px 8px rgb(128 93 25 / 10%);
  transform: none;
}

.issue-action-button:focus-visible {
  outline-color: #d8bc78;
}

.issue-action-button.is-resolved {
  border-color: #bfdfcd;
  --button-start: #eff8f3;
  --button-end: #e4f3e9;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  color: #277047;
  box-shadow: 0 2px 4px rgb(22 101 52 / 6%);
}

.issue-action-button.is-resolved:hover {
  --button-start: #e4f3f4;
  --button-end: #dceee5;
  background: linear-gradient(135deg, var(--button-start), var(--button-end));
  box-shadow: 0 4px 8px rgb(22 101 52 / 10%);
}

.issue-action-button.is-resolved:focus-visible {
  outline-color: #86efac;
}

.issue-action-button:active {
  box-shadow: none;
}

.project-card .issue-description {
  margin: 8px 0 0;
  color: #5f716c;
  font-size: 13px;
  line-height: 1.8;
  white-space: pre-wrap;
}

.project-overview {
  padding: 4px 0 0;
}

.project-meta,
.issues-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
}

.project-label {
  color: #64748b;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 1px;
}

.project-overview h3 {
  margin-top: 12px;
  font-size: 24px;
  line-height: 1.4;
}

.project-overview p {
  margin-bottom: 0;
}

.issues-toolbar {
  margin-bottom: 18px;
}

.issue-count {
  padding: 1px 7px;
  border-radius: 6px;
  background: #e3ebe7;
  color: #52685f;
  font-size: 12px;
  font-variant-numeric: tabular-nums;
}

.project-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  padding-top: 20px;
  border-top: 1px solid #e8edf2;
}

.project-footer-label {
  color: #64748b;
  font-size: 12px;
}

.project-form > .edit-actions {
  flex-wrap: wrap;
}

@media (max-width: 480px) {
  .project-card { padding: 20px 16px; }
  .project-issues { padding: 16px 12px; }
  .issue-form { padding: 16px 12px; }
  .project-actions { gap: 8px; }
}
@media (prefers-reduced-motion: reduce) {
.status-button,
  .issue-status,
  .project-card {
    transition: none;
  }
}

.practice-collapse {
  display: grid;
  grid-template-rows: 1fr;
}

.practice-collapse-inner {
  min-height: 0;
  overflow: hidden;
}

.practice-collapse-enter-active,
.practice-collapse-leave-active {
  transition: grid-template-rows 0.4s ease, opacity 0.4s ease;
}

/* 从编辑框打开起关闭滚动锚定，避免收起开始前已选中下方锚点。 */
:global(html:has(.practice-collapse)) {
  overflow-anchor: none;
}

.practice-collapse-enter-from,
.practice-collapse-leave-to {
  grid-template-rows: 0fr;
  opacity: 0;
}

@media (prefers-reduced-motion: reduce) {
  .practice-collapse-enter-active,
  .practice-collapse-leave-active {
    transition: none;
  }
}

.surface-enter-active,
.surface-leave-active {
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.surface-enter-from,
.surface-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

@media (prefers-reduced-motion: reduce) {
  .surface-enter-active,
  .surface-leave-active {
    transition: none;
  }
}
</style>
