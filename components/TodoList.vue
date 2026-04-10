<template>
  <div class="min-h-screen bg-gray-50 py-8">
    <div class="max-w-4xl mx-auto px-4">
      <!-- Header -->
      <div class="bg-white rounded-lg shadow-sm p-6 mb-6">
        <div class="flex items-center justify-between">
          <div>
            <h1 class="text-3xl font-bold text-gray-900 mb-2">
              📋 Daily Development Workflow
            </h1>
            <p class="text-gray-600">
              Complete development cycle from requirements to testing
            </p>
          </div>
          <div class="text-right">
            <div class="text-sm text-gray-500 mb-1">Progress</div>
            <div class="text-2xl font-bold text-blue-600">
              {{ completedTasks }}/{{ totalTasks }}
            </div>
            <div class="w-32 bg-gray-200 rounded-full h-2 mt-2">
              <div
                class="bg-blue-600 h-2 rounded-full transition-all duration-300"
                :style="{ width: progressPercentage + '%' }"
              ></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Progress Overview -->
      <div class="grid grid-cols-1 md:grid-cols-5 gap-4 mb-6">
        <div
          v-for="phase in phases"
          :key="phase.id"
          class="bg-white rounded-lg shadow-sm p-4"
        >
          <div class="flex items-center justify-between mb-2">
            <h3 class="font-semibold text-gray-900">{{ phase.title }}</h3>
            <span
              class="px-2 py-1 text-xs rounded-full"
              :class="phase.statusClass"
            >
              {{ phase.status }}
            </span>
          </div>
          <div class="text-sm text-gray-600 mb-2">{{ phase.time }}</div>
          <div class="text-xs text-gray-500">
            {{ phase.completed }}/{{ phase.total }} tasks
          </div>
          <div class="w-full bg-gray-200 rounded-full h-1 mt-2">
            <div
              class="h-1 rounded-full transition-all duration-300"
              :class="phase.progressClass"
              :style="{ width: phase.progress + '%' }"
            ></div>
          </div>
        </div>
      </div>

      <!-- Todo Sections -->
      <div class="space-y-6">
        <div
          v-for="section in sections"
          :key="section.id"
          class="bg-white rounded-lg shadow-sm overflow-hidden"
        >
          <!-- Section Header -->
          <div
            class="px-6 py-4 border-b border-gray-200 cursor-pointer hover:bg-gray-50 transition-colors"
            @click="toggleSection(section.id)"
          >
            <div class="flex items-center justify-between">
              <div class="flex items-center space-x-3">
                <span class="text-2xl">{{ section.icon }}</span>
                <div>
                  <h2 class="text-xl font-semibold text-gray-900">
                    {{ section.title }}
                  </h2>
                  <p class="text-sm text-gray-600">{{ section.description }}</p>
                </div>
              </div>
              <div class="flex items-center space-x-2">
                <span class="text-sm text-gray-500">
                  {{ section.completedTasks }}/{{ section.totalTasks }}
                </span>
                <svg
                  class="w-5 h-5 text-gray-400 transform transition-transform"
                  :class="{ 'rotate-180': expandedSections.includes(section.id) }"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path>
                </svg>
              </div>
            </div>
          </div>

          <!-- Section Content -->
          <div
            v-show="expandedSections.includes(section.id)"
            class="px-6 py-4"
          >
            <div class="space-y-3">
              <div
                v-for="task in section.tasks"
                :key="task.id"
                class="flex items-start space-x-3 p-3 rounded-lg hover:bg-gray-50 transition-colors"
              >
                <input
                  :id="task.id"
                  type="checkbox"
                  :checked="task.completed"
                  @change="toggleTask(task.id)"
                  class="mt-1 h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
                />
                <div class="flex-1">
                  <label
                    :for="task.id"
                    class="text-sm font-medium text-gray-900 cursor-pointer"
                    :class="{ 'line-through text-gray-500': task.completed }"
                  >
                    {{ task.title }}
                  </label>
                  <ul
                    v-if="task.subtasks && task.subtasks.length > 0"
                    class="mt-2 ml-4 space-y-1"
                  >
                    <li
                      v-for="subtask in task.subtasks"
                      :key="subtask.id"
                      class="flex items-center space-x-2"
                    >
                      <input
                        :id="subtask.id"
                        type="checkbox"
                        :checked="subtask.completed"
                        @change="toggleSubtask(subtask.id)"
                        class="h-3 w-3 text-green-600 focus:ring-green-500 border-gray-300 rounded"
                      />
                      <label
                        :for="subtask.id"
                        class="text-xs text-gray-600 cursor-pointer"
                        :class="{ 'line-through text-gray-400': subtask.completed }"
                      >
                        {{ subtask.title }}
                      </label>
                    </li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Success Criteria -->
      <div class="bg-white rounded-lg shadow-sm p-6 mt-6">
        <h3 class="text-lg font-semibold text-gray-900 mb-4">🎯 Success Criteria</h3>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div class="flex items-center space-x-2">
            <input
              id="requirements-met"
              type="checkbox"
              :checked="successCriteria.requirements"
              @change="successCriteria.requirements = $event.target.checked"
              class="h-4 w-4 text-green-600 focus:ring-green-500 border-gray-300 rounded"
            />
            <label for="requirements-met" class="text-sm text-gray-700 cursor-pointer">
              Requirements Met
            </label>
          </div>
          <div class="flex items-center space-x-2">
            <input
              id="quality-assurance"
              type="checkbox"
              :checked="successCriteria.quality"
              @change="successCriteria.quality = $event.target.checked"
              class="h-4 w-4 text-green-600 focus:ring-green-500 border-gray-300 rounded"
            />
            <label for="quality-assurance" class="text-sm text-gray-700 cursor-pointer">
              Quality Assurance
            </label>
          </div>
          <div class="flex items-center space-x-2">
            <input
              id="ready-for-review"
              type="checkbox"
              :checked="successCriteria.review"
              @change="successCriteria.review = $event.target.checked"
              class="h-4 w-4 text-green-600 focus:ring-green-500 border-gray-300 rounded"
            />
            <label for="ready-for-review" class="text-sm text-gray-700 cursor-pointer">
              Ready for Review
            </label>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Reactive data
const expandedSections = ref(['requirements'])
const successCriteria = ref({
  requirements: false,
  quality: false,
  review: false
})

// Todo sections data
const sections = ref([
  {
    id: 'requirements',
    title: 'Phase 1: Requirements Analysis',
    description: '9:00 AM - 10:00 AM',
    icon: '🏁',
    tasks: [
      {
        id: 'analyze-requirements',
        title: 'Analyze Requirements',
        completed: false,
        subtasks: [
          { id: 'parse-breakdown', title: 'Parse and break down requirements into atomic items', completed: false },
          { id: 'identify-implicit', title: 'Identify implicit requirements and dependencies', completed: false },
          { id: 'highlight-ambiguities', title: 'Highlight ambiguities and incomplete specifications', completed: false },
          { id: 'flag-conflicts', title: 'Flag conflicting requirements', completed: false }
        ]
      },
      {
        id: 'technical-assessment',
        title: 'Technical Assessment',
        completed: false,
        subtasks: [
          { id: 'evaluate-feasibility', title: 'Evaluate feasibility in Nuxt/Vue 3 ecosystem', completed: false },
          { id: 'propose-approaches', title: 'Propose implementation approaches', completed: false },
          { id: 'identify-risks', title: 'Identify potential risks and edge cases', completed: false },
          { id: 'suggest-alternatives', title: 'Suggest alternative solutions', completed: false }
        ]
      },
      {
        id: 'validation-documentation',
        title: 'Validation & Documentation',
        completed: false,
        subtasks: [
          { id: 'ask-questions', title: 'Ask clarifying questions for ambiguities', completed: false },
          { id: 'create-criteria', title: 'Create acceptance criteria', completed: false },
          { id: 'generate-checklist', title: 'Generate implementation checklist', completed: false },
          { id: 'document-rationale', title: 'Document decision rationale', completed: false }
        ]
      }
    ]
  },
  {
    id: 'implementation',
    title: 'Phase 2: Feature Implementation',
    description: '10:00 AM - 2:00 PM',
    icon: '💻',
    tasks: [
      {
        id: 'architecture-design',
        title: 'Architecture Design',
        completed: false,
        subtasks: [
          { id: 'design-structure', title: 'Design component structure and data flow', completed: false },
          { id: 'plan-integration', title: 'Plan API integration points', completed: false },
          { id: 'consider-performance', title: 'Consider performance implications', completed: false },
          { id: 'define-state', title: 'Define state management approach', completed: false }
        ]
      },
      {
        id: 'core-implementation',
        title: 'Core Implementation',
        completed: false,
        subtasks: [
          { id: 'create-components', title: 'Create Vue 3 components with Composition API', completed: false },
          { id: 'implement-logic', title: 'Implement business logic and data handling', completed: false },
          { id: 'add-error-handling', title: 'Add proper error handling', completed: false },
          { id: 'integrate-codebase', title: 'Integrate with existing codebase', completed: false }
        ]
      },
      {
        id: 'ui-development',
        title: 'UI/UX Development',
        completed: false,
        subtasks: [
          { id: 'responsive-design', title: 'Implement responsive design with Tailwind CSS', completed: false },
          { id: 'accessibility', title: 'Add proper accessibility features', completed: false },
          { id: 'loading-states', title: 'Handle loading states and user feedback', completed: false },
          { id: 'cross-browser', title: 'Test cross-browser compatibility', completed: false }
        ]
      },
      {
        id: 'code-quality',
        title: 'Code Quality',
        completed: false,
        subtasks: [
          { id: 'follow-conventions', title: 'Follow project conventions and standards', completed: false },
          { id: 'add-comments', title: 'Add comprehensive comments and JSDoc', completed: false },
          { id: 'typescript-types', title: 'Ensure TypeScript types are correct', completed: false },
          { id: 'remove-debug', title: 'Remove console logs and debug code', completed: false }
        ]
      }
    ]
  },
  {
    id: 'testing',
    title: 'Phase 3: Testing & Quality Assurance',
    description: '2:00 PM - 4:00 PM',
    icon: '🧪',
    tasks: [
      {
        id: 'test-planning',
        title: 'Test Planning',
        completed: false,
        subtasks: [
          { id: 'identify-paths', title: 'Identify critical paths requiring coverage', completed: false },
          { id: 'plan-cases', title: 'Plan test cases for normal and edge scenarios', completed: false },
          { id: 'define-mocking', title: 'Define mocking strategies for dependencies', completed: false },
          { id: 'set-targets', title: 'Set coverage targets', completed: false }
        ]
      },
      {
        id: 'unit-test-implementation',
        title: 'Unit Test Implementation',
        completed: false,
        subtasks: [
          { id: 'write-function-tests', title: 'Write tests for functions and composables', completed: false },
          { id: 'test-components', title: 'Test Vue components with proper mounting', completed: false },
          { id: 'handle-async', title: 'Handle async operations and promises', completed: false },
          { id: 'setup-teardown', title: 'Implement proper setup/teardown', completed: false }
        ]
      },
      {
        id: 'test-quality',
        title: 'Test Quality',
        completed: false,
        subtasks: [
          { id: 'focused-tests', title: 'Ensure tests are focused and isolated', completed: false },
          { id: 'appropriate-assertions', title: 'Use appropriate assertions and matchers', completed: false },
          { id: 'aaa-pattern', title: 'Follow AAA pattern (Arrange, Act, Assert)', completed: false },
          { id: 'document-logic', title: 'Document complex test logic', completed: false }
        ]
      },
      {
        id: 'coverage-verification',
        title: 'Coverage Verification',
        completed: false,
        subtasks: [
          { id: 'run-suite', title: 'Run test suite and verify all tests pass', completed: false },
          { id: 'check-reports', title: 'Check coverage reports and identify gaps', completed: false },
          { id: 'add-missing', title: 'Add missing test cases as needed', completed: false },
          { id: 'optimize-performance', title: 'Optimize test performance', completed: false }
        ]
      }
    ]
  },
  {
    id: 'review',
    title: 'Phase 4: Code Review & Validation',
    description: '4:00 PM - 5:00 PM',
    icon: '🔍',
    tasks: [
      {
        id: 'code-quality-review',
        title: 'Code Quality Review',
        completed: false,
        subtasks: [
          { id: 'evaluate-structure', title: 'Evaluate code structure and readability', completed: false },
          { id: 'check-naming', title: 'Check naming conventions and consistency', completed: false },
          { id: 'identify-patterns', title: 'Identify anti-patterns and improvements', completed: false },
          { id: 'verify-api', title: 'Verify Vue 3 Composition API usage', completed: false }
        ]
      },
      {
        id: 'security-assessment',
        title: 'Security Assessment',
        completed: false,
        subtasks: [
          { id: 'check-vulnerabilities', title: 'Check for potential vulnerabilities', completed: false },
          { id: 'validate-sanitization', title: 'Validate input sanitization', completed: false },
          { id: 'review-auth', title: 'Review authentication/authorization logic', completed: false },
          { id: 'flag-patterns', title: 'Flag unsafe patterns', completed: false }
        ]
      },
      {
        id: 'performance-analysis',
        title: 'Performance Analysis',
        completed: false,
        subtasks: [
          { id: 'identify-bottlenecks', title: 'Identify performance bottlenecks', completed: false },
          { id: 'review-rendering', title: 'Review rendering optimization', completed: false },
          { id: 'check-leaks', title: 'Check for memory leaks', completed: false },
          { id: 'suggest-caching', title: 'Suggest caching strategies', completed: false }
        ]
      },
      {
        id: 'best-practices',
        title: 'Best Practices Validation',
        completed: false,
        subtasks: [
          { id: 'adhere-standards', title: 'Ensure adherence to project standards', completed: false },
          { id: 'check-dependencies', title: 'Check dependency usage', completed: false },
          { id: 'validate-handling', title: 'Validate error handling', completed: false },
          { id: 'review-annotations', title: 'Review TypeScript annotations', completed: false }
        ]
      }
    ]
  },
  {
    id: 'finalization',
    title: 'Phase 5: Documentation & Finalization',
    description: '5:00 PM - 5:30 PM',
    icon: '📝',
    tasks: [
      {
        id: 'code-documentation',
        title: 'Code Documentation',
        completed: false,
        subtasks: [
          { id: 'update-component-docs', title: 'Update component documentation', completed: false },
          { id: 'add-api-docs', title: 'Add API documentation if needed', completed: false },
          { id: 'document-changes', title: 'Document breaking changes', completed: false },
          { id: 'update-readme', title: 'Update README if required', completed: false }
        ]
      },
      {
        id: 'commit-preparation',
        title: 'Commit Preparation',
        completed: false,
        subtasks: [
          { id: 'write-message', title: 'Write clear commit message', completed: false },
          { id: 'ensure-committed', title: 'Ensure all changes are committed', completed: false },
          { id: 'verify-build', title: 'Verify build passes', completed: false },
          { id: 'check-linting', title: 'Check for any linting errors', completed: false }
        ]
      },
      {
        id: 'final-verification',
        title: 'Final Verification',
        completed: false,
        subtasks: [
          { id: 'run-tests', title: 'Run full test suite one more time', completed: false },
          { id: 'verify-feature', title: 'Verify feature works as expected', completed: false },
          { id: 'check-console', title: 'Check console for errors', completed: false },
          { id: 'test-edge-cases', title: 'Test edge cases manually', completed: false }
        ]
      }
    ]
  }
])

// Computed properties
const totalTasks = computed(() => {
  return sections.value.reduce((total, section) => {
    return total + section.tasks.reduce((sectionTotal, task) => {
      return sectionTotal + 1 + (task.subtasks ? task.subtasks.length : 0)
    }, 0)
  }, 0)
})

const completedTasks = computed(() => {
  return sections.value.reduce((total, section) => {
    return total + section.tasks.reduce((sectionTotal, task) => {
      let count = task.completed ? 1 : 0
      if (task.subtasks) {
        count += task.subtasks.filter(subtask => subtask.completed).length
      }
      return sectionTotal + count
    }, 0)
  }, 0)
})

const progressPercentage = computed(() => {
  return totalTasks.value > 0 ? Math.round((completedTasks.value / totalTasks.value) * 100) : 0
})

const phases = computed(() => {
  return sections.value.map(section => {
    const total = section.tasks.reduce((sum, task) => sum + 1 + (task.subtasks ? task.subtasks.length : 0), 0)
    const completed = section.tasks.reduce((sum, task) => {
      let count = task.completed ? 1 : 0
      if (task.subtasks) {
        count += task.subtasks.filter(subtask => subtask.completed).length
      }
      return sum + count
    }, 0)

    const progress = total > 0 ? Math.round((completed / total) * 100) : 0

    let status = 'Not Started'
    let statusClass = 'bg-gray-100 text-gray-800'
    let progressClass = 'bg-gray-400'

    if (progress === 100) {
      status = 'Completed'
      statusClass = 'bg-green-100 text-green-800'
      progressClass = 'bg-green-600'
    } else if (progress > 0) {
      status = 'In Progress'
      statusClass = 'bg-blue-100 text-blue-800'
      progressClass = 'bg-blue-600'
    }

    return {
      id: section.id,
      title: section.title.replace('Phase ', '').split(':')[0],
      time: section.description,
      status,
      statusClass,
      progress,
      progressClass,
      completed,
      total
    }
  })
})

// Methods
const toggleSection = (sectionId) => {
  const index = expandedSections.value.indexOf(sectionId)
  if (index > -1) {
    expandedSections.value.splice(index, 1)
  } else {
    expandedSections.value.push(sectionId)
  }
}

const toggleTask = (taskId) => {
  sections.value.forEach(section => {
    section.tasks.forEach(task => {
      if (task.id === taskId) {
        task.completed = !task.completed
        // Auto-complete subtasks if main task is completed
        if (task.completed && task.subtasks) {
          task.subtasks.forEach(subtask => {
            subtask.completed = true
          })
        }
        // Auto-uncomplete subtasks if main task is uncompleted
        if (!task.completed && task.subtasks) {
          task.subtasks.forEach(subtask => {
            subtask.completed = false
          })
        }
      }
    })
  })
  saveProgress()
}

const toggleSubtask = (subtaskId) => {
  sections.value.forEach(section => {
    section.tasks.forEach(task => {
      if (task.subtasks) {
        task.subtasks.forEach(subtask => {
          if (subtask.id === subtaskId) {
            subtask.completed = !subtask.completed
            // Auto-complete main task if all subtasks are completed
            const allCompleted = task.subtasks.every(st => st.completed)
            if (allCompleted) {
              task.completed = true
            } else {
              task.completed = false
            }
          }
        })
      }
    })
  })
  saveProgress()
}

const saveProgress = () => {
  if (process.client) {
    localStorage.setItem('todo-progress', JSON.stringify({
      sections: sections.value,
      successCriteria: successCriteria.value,
      expandedSections: expandedSections.value
    }))
  }
}

const loadProgress = () => {
  if (process.client) {
    const saved = localStorage.getItem('todo-progress')
    if (saved) {
      const data = JSON.parse(saved)
      sections.value = data.sections || sections.value
      successCriteria.value = data.successCriteria || successCriteria.value
      expandedSections.value = data.expandedSections || expandedSections.value
    }
  }
}

// Lifecycle
onMounted(() => {
  loadProgress()
})
</script>

<style scoped>
/* Custom styles for better UX */
.rotate-180 {
  transform: rotate(180deg);
}

/* Smooth transitions */
.transition-all {
  transition: all 0.3s ease;
}

.transition-colors {
  transition: color 0.2s ease, background-color 0.2s ease;
}
</style></content>
<parameter name="filePath">c:\Users\Acer\Documents\workshopvux\components\TodoList.vue