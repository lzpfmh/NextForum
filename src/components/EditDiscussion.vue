<template>
  <el-form size="small">
    <div v-if="isEditTopic">
      <el-form-item label="主题" :label-width="labelWidth">
        <el-input class="input-body" placeholder="输入讨论的主题"
          v-model="data.topic"
          :rules="[
            { required: true, message: '主题不能为空', }
          ]"></el-input>
      </el-form-item>
      <!-- <el-form-item label="分类" :label-width="labelWidth">
        <el-cascader
          class="input-body"
          :options="categories"
          @active-item-change="loadSubCategories"
          v-model="data.category">
        </el-cascader>
      </el-form-item> -->
      <el-form-item label="标签（可选）" :label-width="labelWidth">
        <el-select
          class="input-body"
          multiple
          filterable
          v-model="data.tags"
          placeholder="请选择讨论的标签，可多选">
          <el-option
            v-for="tag in tags"
            :key="tag" :label="tag" :value="tag">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label-width="labelWidth">
        <editor ref="editor" :initialValue="data.contents" :onPickEmoji="pickEmoji" />
      </el-form-item>
    </div>
    <div v-if="!isEditTopic">
      <editor ref="editor" :initialValue="data.contents" :onPickEmoji="pickEmoji" />
    </div>
  </el-form>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import MyEditor from './MyEditor'

export default {
  name: 'EditDiscussion',
  props: [
    'post',
    'topic',
  ],
  components: {
    editor: MyEditor,
  },
  data () {
    return {
      labelWidth: '140px',
      categories: [],
      tags: [],
      data: {
        topic: null,
        category: null,
        tags: [],
        contents: '',
      },
      isEditTopic: false,
    }
  },
  computed: {
    ...mapGetters([
      'allTags',
      'allCategories',
    ]),
    categoriesOption() {
      return this.createCategoriesOption(this.allCategories)
    },
    tagsOption() {
      return this.allTags.map(tag => tag.id)
    },
    contents() {
      return this.$refs.editor.value
    },
  },
  methods: {
    ...mapActions([
      'getTags',
      'getCategories',
    ]),
    createCategoriesOption(list, ownChildren = true) {
      return list.map((item) => {
        var option = {
          label: item.name,
          value: item.id,
        }
        if (!item.has_children)
          return option

        option.children = ownChildren ? [] : undefined
        return option
      })
    },
    async loadSubCategories(val) {
      var parentId = val[val.length - 1]
      this.data.category = [parentId]

      var category = this.categories.find(
        item => item.value === parentId)
      if (category) {
        category.children = this.createCategoriesOption(
          await this.getCategories(parentId), false)
      }
    },
    async setPost(item) {
      this.isEditTopic = item.post_number === 1
      var { data } = await this.$http.get(`/posts/${item.id}`);
      this.data.contents = data.raw
    },
    pickEmoji(item) {
      console.log('pick pickEmoji', item)
      this.data.contents += item.native
    },
  },
  async mounted() {
    this.categories = this.categoriesOption
    this.tags = this.tagsOption

    // set data contents
    this.data.topic = this.topic.title
    this.data.category = this.topic.category
    this.data.tags = this.topic.tags
  },
}
</script>

<style scoped>
.input-body {
  width: 100%;
}
</style>