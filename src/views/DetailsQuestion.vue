<template>
  <div class="question-details" v-loading="loading">
    <div class="question-header">
      <!-- el-dialog 弹窗是默认隐藏的 -->
      <el-dialog
        title="修改问题"
        :visible.sync="askModelVisible"
        :modal-append-to-body="false"
      >
        <!-- 事件使用连字符连接的形式 -->
        <ask-model
          @change-ask-model-visible="changeAskModelVisible"
          @update-question="getQuestion"
          :oldItem="questionData"
        />
      </el-dialog>

      <div class="question-header-content">
        <div class="question-header-main">
          <h1 class="question-header-title">
            {{ questionData.title }}
            <el-button
              type="text"
              class="m-l-25 gray"
              @click="askModelVisible = true"
            >
              <i class="el-icon-edit el-icon--left"></i>编辑
            </el-button>
          </h1>

          <div
            class="question-header-details"
            v-show="showType === 'excerpt'"
            v-if="questionData.excerpt"
          >
            <span>{{ questionData.excerpt }}</span>
            <el-button
              class="btn-no-padding m-l-10"
              type="text"
              icon="el-icon-arrow-down"
              @click="showType = 'all'"
            >
              阅读全文
            </el-button>
          </div>
          <div
            class="question-header-details"
            v-show="showType === 'all'"
            v-if="questionData.excerpt"
          >
            <div v-html="questionData.description"></div>
            <el-button
              class="btn-no-padding"
              type="text"
              icon="el-icon-arrow-up"
              @click="showType = 'excerpt'"
            >
              收起
            </el-button>
          </div>
        </div>
        <!-- /.question-header-main -->

        <div class="question-header-side">
          <div class="question-header-side-item">
            <p class="name">被浏览</p>
            <p class="num">232</p>
          </div>
          <div class="question-header-side-item">
            <p class="name">关注者</p>
            <p class="num">12343</p>
          </div>
        </div>
      </div>
      <!-- /.question-header-content -->

      <div class="question-header-footer">
        <div class="question-header-footer-inner">
          <div class="question-header-footer-main">
            <div class="question-header-btnGroup">
              <el-button type="primary">关注问题</el-button>
              <el-button
                type="primary"
                plain
                icon="el-icon-edit"
                @click="showAnswerPart"
                :disabled="!currentAnswerEmpty"
              >
                写回答
              </el-button>
            </div>
            <div class="question-header-actions">
              <el-button class="button" type="info" plain>
                <span class="el el-icon-fakezhihu-add-person-fill"></span
                >邀请回答
              </el-button>
              <list-item-actions
                class="actions"
                :itemId="questionData.id"
                :type="1"
                :showActionItems="['comment', 'share', 'more']"
              />
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 写回答的编辑框，默认隐藏 -->
    <div class="question-main">
      <div class="question-main-clo">
        <el-card
          class="m-b-15"
          v-loading="authorLoading"
          v-show="answerVisible"
        >
          <div class="author-info m-t-25">
            <div class="avatar">
              <img :src="authorInfo.avatarUrl || ''" alt="" />
            </div>
          </div>
          <div class="userinfo">
            <p class="username">{{ authorInfo.name }}</p>
            <p class="headline">{{ authorInfo.headline }}</p>
          </div>
          <rich-text-editor
            class="with-border m-t-25 m-b-25"
            ref="richtext"
            :content="answerContent"
            :placeHolder="placeHolder"
            @update-content="updateContent"
          />
          <div class="m-b-25">
            <el-button type="default" @click="answerVisible = false">
              取消
            </el-button>
            <el-button type="primary" @click="createAnswer">提交回答</el-button>
          </div>
        </el-card>

        <el-card v-if="!currentAnswerEmpty" class="m-b-25">
          <list-item
            class="without-border no-padding"
            :item="currentAnswer"
            :showPart="['creator', 'votes']"
            :type="2"
          />
        </el-card>
        <el-card v-show="allAnswerLength === 0">
          <div class="no-answer m-t-25 m-b-25">当前没有回答</div>
        </el-card>
        <el-card v-show="answersExist">
          <div class="list">
            <div class="list-header">
              <span>
                {{ questionData.answers ? questionData.answers.length : 0 }}
                个回答
              </span>
            </div>
            <div
              class="list-item"
              v-for="(answer, index) in questionData.answers"
              :key="index"
            >
              <list-item
                :item="answer"
                :index="index"
                :showPart="['creator', 'votes']"
                :type="2"
              />
            </div>
          </div>
        </el-card>
      </div>

      <div class="question-main-sidebar">
        <el-card class="m-b-15">
          <div class="header"><span>相关问题</span></div>
          <div class="content">
            <div class="content-item">
              <a href="#" class="question">
                vuex 是什么？怎么使用？哪种功能场景使用它？
              </a>
              <span class="answer-count">244个回答</span>
            </div>
            <div class="content-item">
              <a href="#" class="question">
                vue-router 有哪几种导航钩子?
              </a>
              <span class="answer-count">244个回答</span>
            </div>
            <div class="content-item">
              <a href="#" class="question">
                vue 的双向绑定的原理是什么(常考)？
              </a>
              <span class="answer-count">244个回答</span>
            </div>
          </div>
        </el-card>
        <sidebar-footer />
      </div>
    </div>
  </div>
</template>

<script>
import request from "@/service";
import { getCookies } from "@/lib/utils";
import AskModel from "../components/QuickScore.vue";
import ListItemActions from "../components/ListItemActions.vue";
import RichTextEditor from "../components/RichTextEditor.vue";
import SidebarFooter from "../components/SidebarFooter.vue";
import _ from "lodash";
import ListItem from "../components/ListItem.vue";

export default {
  components: {
    AskModel,
    ListItemActions,
    RichTextEditor,
    SidebarFooter,
    ListItem
  },
  data() {
    return {
      loading: true,
      askModelVisible: false, // 修改问题框是否可见
      questionData: {},
      authorLoading: false,
      answerVisible: false, // 回答框是否可见
      commentShowType: "all", // 评论展示状态
      showType: "excerpt", // 问题详情展示状态
      answerContent: "",
      answerExcerpt: "", // 回答内容简介
      placeHolder: "写回答", // 回答框输入placeholder
      authorInfo: {},
      currentAnswer: {} // 当前作者回答
    };
  },
  mounted() {
    this.getQuestion();
  },
  computed: {
    currentAnswerEmpty() {
      return _.isEmpty(this.currentAnswer);
    },
    allAnswerLength() {
      const questionDataAnswersLength = this.questionData.answers
        ? this.questionData.answers.length
        : 0;
      return this.currentAnswerEmpty
        ? questionDataAnswersLength
        : questionDataAnswersLength + 1;
    },
    answersExist() {
      return this.questionData.answers
        ? this.questionData.answers.length > 0
        : false;
    }
  },
  methods: {
    changeAskModelVisible(status) {
      this.askModelVisible = status;
    },
    async getQuestion() {
      this.loading = true;
      await request
        .get("/questions", {
          questionId: this.$route.params.id
        })
        .then(res => {
          this.questionData = res.data.content;
          this.questionData.answers = _.compact(
            _.map(this.questionData.answers, item => {
              if (item.creatorId === parseFloat(getCookies("id"))) {
                this.currentAnswer = item;
                return null;
              }
              return item;
            })
          );
          this.loading = false;
        });
    },
    async getAuthorInfo() {
      this.authorLoading = true;
      await request
        .get("/users", {
          userId: getCookies("id")
        })
        .then(res => {
          if (res.data.status === 200) {
            this.authorInfo = res.data.content;
            this.authorLoading = false;
          }
        });
    },
    async createAnswer() {
      this.authorLoading = true;
      await request
        .post("/answers", {
          creatorId: getCookies("id"),
          content: this.answerContent,
          excerpt: this.answerExcerpt,
          targetId: this.questionData.id
        })
        .then(res => {
          if (res.data.status === 201) {
            this.$message.success("回答成功");
            this.authorLoading = false;
            this.answerVisible = false;
            this.getQuestion();
          } else {
            this.$message.error("回答失败，请稍后再试");
          }
        });
    },
    updateContent(content, contentText) {
      this.answerContent = content;
      this.answerExcerpt = contentText.slice(0, 100);
    },
    showAnswerPart() {
      this.answerVisible = true;
      this.getAuthorInfo();
    }
  }
};
</script>

<style></style>
