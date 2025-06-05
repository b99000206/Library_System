<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import {
  AlertDialog,
  AlertDialogTrigger,
  AlertDialogContent,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogCancel,
  AlertDialogAction
} from '@/components/ui/alert-dialog'
import { Icon } from '#components'

const route = useRoute()
const router = useRouter()
const search = ref('')

watch(
  () => route.query.search,
  (keyword) => {
    if (typeof keyword === 'string') {
      search.value = keyword
    }
  },
  { immediate: true }
)

const tab = ref<'profile' | 'records'>('profile')

const member = ref({
  name: '張小明',
  email: 'xiaoming@example.com',
  studentId: 'A123456789',
  phone: '0912345678',
  birthday: '2000-01-01',
  password: '********'
})

const borrowRecords = ref([
  {
    title: '解憂雜貨店',
    author: '東野圭吾',
    date: '2025-05-10',
    cover: 'https://upload.wikimedia.org/wikipedia/zh/thumb/0/0b/Namiya.jpg/220px-Namiya.jpg'
  },
  {
    title: '追風箏的人',
    author: '卡勒德·胡賽尼',
    date: '2025-04-22',
    cover: 'https://example.com/kite.jpg'
  }
])

const sortBy = ref<'borrowDate' | 'reviewDate'>('borrowDate')

const reviewMap = ref<Record<string, {
  content: string
  rating: number
  createdAt: string
  updatedAt?: string
}>>({})

const sortedRecords = computed(() => {
  return [...borrowRecords.value].sort((a, b) => {
    if (sortBy.value === 'borrowDate') {
      return new Date(b.date).getTime() - new Date(a.date).getTime()
    }
    const reviewA = reviewMap.value[a.title]
    const reviewB = reviewMap.value[b.title]
    const dateA = reviewA?.updatedAt || reviewA?.createdAt || ''
    const dateB = reviewB?.updatedAt || reviewB?.createdAt || ''
    return new Date(dateB).getTime() - new Date(dateA).getTime()
  })
})

const currentReviewTitle = ref('')
const showDialog = ref(false)
const showReviewForm = ref(false)
const reviewContent = ref('')
const rating = ref(5)
const isEditing = ref(false)

function confirmWriteReview(title: string) {
  currentReviewTitle.value = title
  showDialog.value = true
  showReviewForm.value = false
}

function proceedToWriteReview() {
  showDialog.value = false
  showReviewForm.value = true
  const existing = reviewMap.value[currentReviewTitle.value]
  if (existing) {
    reviewContent.value = existing.content
    rating.value = existing.rating
    isEditing.value = true
  } else {
    reviewContent.value = ''
    rating.value = 5
    isEditing.value = false
  }
}

function submitReview() {
  if (!reviewContent.value.trim()) {
    alert('請輸入心得內容')
    return
  }
  const now = new Date().toISOString()
  reviewMap.value[currentReviewTitle.value] = {
    content: reviewContent.value,
    rating: rating.value,
    createdAt: isEditing.value
      ? reviewMap.value[currentReviewTitle.value].createdAt
      : now,
    updatedAt: isEditing.value ? now : undefined
  }
  alert(`${isEditing.value ? '已更新' : '已提交'}「${currentReviewTitle.value}」的心得：\n${reviewContent.value}（${rating.value}分）`)
  reviewContent.value = ''
  rating.value = 5
  showReviewForm.value = false
  isEditing.value = false
}

function deleteReview(title: string) {
  if (confirm(`確定要刪除「${title}」的心得嗎？`)) {
    delete reviewMap.value[title]
    if (currentReviewTitle.value === title) {
      showReviewForm.value = false
    }
  }
}


function goToAuthorSearch(author: string) {
  router.push({ path: '/comment/experience', query: { search: author } })
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}
</script>


<template>
  <div class="p-4 max-w-xl space-y-6">
    <h1 class="text-2xl font-bold mb-4">👤 會員資料管理</h1>

    <div class="flex gap-4">
      <button
        :class="tab === 'profile' ? 'text-blue-600 font-semibold' : 'text-gray-600'"
        @click="tab = 'profile'"
      >修改個人資料</button>
      <button
        :class="tab === 'records' ? 'text-blue-600 font-semibold' : 'text-gray-600'"
        @click="tab = 'records'"
      >借閱紀錄</button>
    </div>

    <div v-if="tab === 'profile'" class="space-y-4">
      <div v-for="(val, key) in member" :key="key">
        <label class="block text-sm font-medium mb-1">{{ key }}</label>
        <input v-model="member[key]" class="w-full border px-3 py-2 rounded" />
      </div>
      <button class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">儲存變更</button>
    </div>

    <div v-else class="space-y-6">
      <h2 class="text-lg font-semibold">📚 借閱紀錄</h2>
      <div class="flex items-center gap-2 text-sm">
        <label for="sort">排序方式：</label>
        <select id="sort" v-model="sortBy" class="border rounded px-2 py-1">
          <option value="borrowDate">依借閱日期</option>
          <option value="reviewDate">依心得撰寫日期</option>
        </select>
      </div>

      <ul class="space-y-6 text-gray-700">
        <li v-for="record in sortedRecords" :key="record.title" class="border p-4 rounded space-y-2">
          <div class="flex items-center gap-4">
            <img :src="record.cover" alt="封面" class="w-16 h-auto rounded shadow" />
            <div class="flex-1">
              <div class="font-medium">
                {{ record.title }}
                <span class="text-sm text-blue-600 underline cursor-pointer ml-1" @click="goToAuthorSearch(record.author)">
                  ({{ record.author }})
                </span>
              </div>
              <div class="text-sm text-gray-600">借閱日期：{{ record.date }}</div>
              <button @click="confirmWriteReview(record.title)" class="text-sm text-blue-600 underline mt-1">
                {{ reviewMap[record.title] ? '編輯心得' : '撰寫心得' }}
              </button>
            </div>
          </div>

          <div v-if="showReviewForm && currentReviewTitle === record.title" class="space-y-3">
            <div class="flex items-center space-x-1">
              <span class="text-sm font-medium">評分：</span>
              <div class="flex">
                <button
                  v-for="star in 5"
                  :key="star"
                  @click="rating = star"
                  class="hover:scale-110 transition-transform"
                >
                  <Icon
                    :name="'i-lucide-star'"
                    :class="star <= rating ? 'text-yellow-500' : 'text-gray-300'"
                  />
                </button>
              </div>
              <span class="text-sm ml-2">{{ rating }}/5</span>
            </div>

            <textarea
              v-model="reviewContent"
              class="w-full border px-3 py-2 rounded min-h-[120px]"
              placeholder="輸入您的心得..."
            ></textarea>

            <div class="flex gap-2">
              <button
                @click="submitReview"
                class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700"
              >
                {{ isEditing ? '更新心得' : '提交心得' }}
              </button>
              <button
                v-if="isEditing"
                @click="deleteReview(currentReviewTitle)"
                class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600"
              >
                刪除心得
              </button>
            </div>
          </div>

          <div v-if="reviewMap[record.title]" class="bg-gray-50 border p-3 rounded space-y-1">
            <div class="text-sm font-medium">心得內容：</div>
            <div class="text-sm">{{ reviewMap[record.title].content }}</div>
            <div class="text-sm text-gray-600">
              評分：
              <span v-for="star in 5" :key="star">
                <Icon
                  :name="'i-lucide-star'"
                  :class="star <= reviewMap[record.title].rating ? 'text-yellow-500' : 'text-gray-300'"
                />
              </span>
              （{{ reviewMap[record.title].rating }}分）
            </div>
            <div class="text-xs text-gray-400">
              發表於：{{ reviewMap[record.title].createdAt.slice(0, 10) }}
              <span v-if="reviewMap[record.title].updatedAt">（已編輯）</span>
            </div>
          </div>
        </li>
      </ul>
    </div>

    <!-- 回到頂端按鈕 -->
    <div class="fixed bottom-6 right-6">
      <button
        @click="scrollToTop"
        class="bg-blue-500 text-white px-4 py-2 rounded-full shadow-lg hover:bg-blue-600"
      >回到頂端 ↑</button>
    </div>
  </div>

  <AlertDialog :open="showDialog" @update:open="showDialog = $event">
    <AlertDialogContent>
      <AlertDialogHeader>
        <AlertDialogTitle>📢 撰寫心得前請注意</AlertDialogTitle>
        <AlertDialogDescription class="space-y-2">
          <p>1. 請勿使用辱罵性或攻擊性言語</p>
          <p>2. 禁止張貼任何形式的廣告</p>
          <p>3. 發表內容須與書籍相關，請勿離題</p>
        </AlertDialogDescription>
      </AlertDialogHeader>
      <AlertDialogFooter>
        <AlertDialogCancel>取消</AlertDialogCancel>
        <AlertDialogAction @click="proceedToWriteReview">我同意</AlertDialogAction>
      </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>
