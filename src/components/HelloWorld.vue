<template>
  <el-tabs
    v-model="activeTab"
    @tab-click="handleTabClick"
    type="card"
    class="tabs"
  >
    <el-tab-pane label="当前页面" name="current">
      <div class="container">
        <el-button @click="clearForm" type="info" class="button">
          清空表单
        </el-button>
        <el-select
          v-model="selectedBook"
          placeholder="请选择词书"
          class="select"
        >
          <el-option
            v-for="book in bookList"
            :key="book.ciBookId"
            :label="book.bookName"
            :value="book.ciBookId"
          ></el-option>
        </el-select>
        <div class="word-container">
          <el-input
            v-model="word.term"
            placeholder="词汇"
            class="input"
          ></el-input>
          <div
            v-for="(translation, translationIndex) in word.translations"
            :key="translationIndex"
            class="translation-container"
          >
            <el-input
              v-model="translation.translation"
              placeholder="翻译"
              class="input"
            ></el-input>
            <div class="button-container">
              <el-button
                v-if="translationIndex === word.translations.length - 1"
                @click="addTranslation"
                type="primary"
                class="add-button"
              >
                +
              </el-button>
              <el-button
                v-if="word.translations.length > 1"
                @click="deleteTranslation(translationIndex)"
                type="danger"
                class="delete-button"
              >
                -
              </el-button>
            </div>
          </div>
          <div
            v-for="(sentence, sentenceIndex) in word.sentences"
            :key="sentenceIndex"
            class="sentence-container"
          >
            <el-input
              v-model="sentence.sentenceBurma"
              placeholder="缅甸语例句"
              class="input"
            ></el-input>
            <el-input
              v-model="sentence.sentenceChinese"
              placeholder="中文例句"
              class="input"
            ></el-input>
            <div class="button-container">
              <el-button
                v-if="sentenceIndex === word.sentences.length - 1"
                @click="addSentence"
                type="primary"
                class="add-button"
              >
                +
              </el-button>
              <el-button
                v-if="word.sentences.length > 1"
                @click="deleteSentence(sentenceIndex)"
                type="danger"
                class="delete-button"
              >
                -
              </el-button>
            </div>
          </div>
        </div>
        <el-button @click="saveData" type="primary" class="button">
          保存数据
        </el-button>
      </div>
    </el-tab-pane>
    <el-tab-pane label="新建词书" name="newBook">
      <!-- 新建词书页面内容 -->
      <div class="container">
        <el-input
          v-model="ciBookName"
          placeholder="请输入词书名称"
          class="input"
        ></el-input>
        <el-button @click="createCiBook" type="primary" class="button">
          新建词书
        </el-button>
      </div>
    </el-tab-pane>
    <el-tab-pane label="词汇列表" name="wordList">
      <el-table :data="treeData" style="width: 100%;" row-key="id" border>
        <el-table-column
          prop="bookName"
          label="词书名称"
          width="200"
        ></el-table-column>
        <el-table-column label="词汇">
          <template v-slot="{ row }">
            <div
              v-for="vocab in row.vocabularies"
              :key="vocab.id"
              class="vocab-container"
            >
              <div class="vocab-item">
                词汇名称：{{ vocab.vocabularyChars }}
              </div>
              <div class="example-translation">
                <ul v-if="vocab.children.exampleSentences.length > 0">
                  <li
                    v-for="sentence in vocab.children.exampleSentences"
                    :key="sentence.id"
                  >
                    中文例句：{{ sentence.sentenceChinese }} - 缅语例句{{
                      sentence.sentenceBurmese
                    }}
                  </li>
                </ul>
                <ul v-if="vocab.children.translations.length > 0">
                  <li
                    v-for="translation in vocab.children.translations"
                    :key="translation.id"
                  >
                    翻译：{{ translation.translationText }}
                  </li>
                </ul>
              </div>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-tab-pane>
  </el-tabs>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import {
  ElTabs,
  ElTabPane,
  ElSelect,
  ElOption,
  ElInput,
  ElButton,
  ElMessage,
} from 'element-plus'
import axios from 'axios'

const activeTab = ref('current')
const selectedBook = ref(1)
const ciBookName = ref('')
const bookList = ref([])
const treeData = ref([])

const word = ref({
  term: '',
  translations: [{ translation: '' }],
  sentences: [{ sentenceBurma: '', sentenceChinese: '' }],
})

const addTranslation = () => {
  word.value.translations.push({ translation: '' })
}

const addSentence = () => {
  word.value.sentences.push({ sentenceBurma: '', sentenceChinese: '' })
}

const deleteTranslation = (translationIndex) => {
  word.value.translations.splice(translationIndex, 1)
}

const deleteSentence = (sentenceIndex) => {
  word.value.sentences.splice(sentenceIndex, 1)
}

const createCiBook = () => {
  axios
    .post('/api/v1/book/create', { ciBookName: ciBookName.value })
    .then((response) => {
      if (response.code == 200) {
        ElMessage({
          message: `新建词书${ciBookName.value}成功！`,
          type: 'success',
        })
        getTreeData()
      } else {
        ElMessage.error('新建词书失败！')
      }
      console.log('Book created successfully:', response.data)
    })
    .catch((error) => {
      ElMessage.error('新建词书失败！')
    })
}
const getTreeData = () => {
  axios
    .get('/api/v1/vocabulary/list')
    .then((response) => {
      treeData.value = response.data
      console.log('Tree data retrieved successfully:', response.data)
    })
    .catch((error) => {
      console.error('Error retrieving tree data:', error)
    })
}

const saveData = () => {
  const obj = {
    ciBook: selectedBook.value,
    vocabulary: word.value.term,
    translations: word.value.translations.map(
      (translation) => translation.translation,
    ),
    exampleSentences: word.value.sentences.map((sentence) => {
      return {
        chinese: sentence.sentenceChinese,
        burmese: sentence.sentenceBurma,
      }
    }),
  }

  const bookId = selectedBook.value

  axios
    .post(`/api/v1/vocabulary/create`, obj)
    .then((response) => {
      if (response.data.code == 200) {
        ElMessage({
          message: `新建词汇${obj.vocabulary}成功！`,
          type: 'success',
        })
        getTreeData()
      } else {
        ElMessage.error('新建词书失败！')
      }
      console.log('Data saved successfully:', response.data)
    })
    .catch((error) => {
      console.error('Error saving data:', error)
    })
}

const handleTabClick = (tab) => {
  // 处理标签页切换事件
}

const clearForm = () => {
  selectedBook.value = 1
  ciBookName.value = ''
  word.value.term = ''
  word.value.translations = [{ translation: '' }]
  word.value.sentences = [{ sentenceBurma: '', sentenceChinese: '' }]
}

onMounted(() => {
  axios
    .get('/api/v1/book/list')
    .then((response) => {
      bookList.value = response.data
      console.log('Book list retrieved successfully:', response.data)
    })
    .catch((error) => {
      console.error('Error retrieving book list:', error)
    })
  getTreeData()
})
</script>

<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.select {
  margin-bottom: 20px;
}

.word-container {
  margin-bottom: 20px;
}

.input {
  width: 100%;
  margin-bottom: 10px;
}

.translation-container,
.sentence-container {
  display: flex;
  align-items: center;
}

.button-container {
  margin-left: 10px;
}

.add-button,
.delete-button {
  cursor: pointer;
}

.tabs {
  width: 1000px;
  margin-top: 20px;
}
</style>
