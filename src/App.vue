<script setup lang="ts">
import { reactive, ref, watch } from 'vue'
interface Project {
  id: number
  name: string
  goal: string
  status: '进行中' | '已完成'
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

watch(projects, () => {
  try {
    localStorage.setItem('devtrail-projects', JSON.stringify(projects))
    storageError.value = ''
  } catch {
    storageError.value = '保存项目失败，当前修改可能在刷新后丢失。'
  }
})

const newProjectName = ref('')
const newProjectGoal = ref('')
const formError = ref('')
const editingProjectId = ref<number | null>(null)
const editingProjectName = ref('')
const editingProjectGoal = ref('')
const editError = ref('')

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
    projects.splice(index, 1)
    if (editingProjectId.value === project.id) {
      cancelEdit()
    }
  }
}
</script>

<template>
  <main class="home">
    <header class="page-header">
      <h1>{{ appName }}</h1>
      <p>{{ description }}</p>
    </header>

    <section>
      <h2>我的学习项目</h2>

      <p v-if="storageError" class="form-error" role="alert">
        {{ storageError }}
      </p>

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
        <p v-if="formError" class="form-error" role="alert">
          {{ formError }}
        </p>
        <button type="submit" class="status-button">
          新增项目
        </button>
      </form>

      <p v-if="projects.length === 0">
        还没有学习项目，请通过上方表单新增项目。
      </p>

      <article 
      v-for="project in projects"
      :key="project.id"
      class="project-card"
      >
        <span class="status">{{ project.status }}</span>
        <h3>{{ project.name }}</h3>
        <p>{{ project.goal }}</p>
        
        <form
          v-if="editingProjectId === project.id"
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
          <p v-if="editError" class="form-error" role="alert">
            {{ editError }}
          </p>
          <div class="edit-actions">
            <button type="submit" class="status-button save-button">
              保存
            </button>
            <button type="button" class="status-button cancel-button" @click="cancelEdit">
              取消
            </button>
          </div>
        </form>
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
      </article>
    </section>
  </main>
</template>

<style scoped>
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
  background: linear-gradient(135deg, #ddf4f5, #dbf1e6);
  color: #166534;
  font: inherit;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 2px 4px rgb(22 101 52 / 6%);
  transition: box-shadow 0.2s, transform 0.2s;
}

.project-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.status-button:hover {
  background: linear-gradient(135deg, #bbf7d0, #86efac);
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
.project-form textarea {
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
.project-form textarea:focus-visible {
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

.project-actions > .status-button:hover {
  background: linear-gradient(135deg, #e4f3f4, #dceee5);
}

.save-button,
.cancel-button {
  border-color: #ddd2ed;
  background: linear-gradient(135deg, #f5f0fb, #ece3f7);
  color: #685080;
  box-shadow: 0 2px 4px rgb(104 80 128 / 6%);
}

.save-button:hover,
.cancel-button:hover {
  background: linear-gradient(135deg, #eee5f8, #e5d8f2);
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
</style>
