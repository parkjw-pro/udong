<template>
  <div>
    <!-- <b-card-group deck> -->
    <b-card>
      <!-- 1. 상단 부분 -->
      <b-card-text>
        <b-row align-h="justify" style="text-align: left;">
          <b-col align-self="center">
            <span style="cursor: pointer;" @click="toFeed">
              <!-- 뱃지 -->
              <b-avatar :src="require(`@/assets/app/badge/${this.post.isOpen}.png`)"></b-avatar>
              <!-- 닉네임 -->
              <span class="ml-1" style="">{{ post.nickname }}</span>
            </span>
          </b-col>
          <!-- 추가 버튼(신고/삭제) -->
          <b-col style="text-align: right;">
            <b-dropdown
              size="lg"
              dropup
              variant="link"
              toggle-class="text-decoration-none"
              no-caret
            >
              <template #button-content>
                <b-icon icon="three-dots" variant="dark"></b-icon>
              </template>
              <div v-if="post.userId === userId">
                <b-dropdown-item href="" variant="danger" @click="showModal">삭제</b-dropdown-item>
                <!-- 삭제 modal 창 -->

                <b-modal :ref="post.postId" @ok="deletePost">
                  <p>
                    <img alt="Vue logo" src="@/assets/udonge.png" style="width: 10%" />소중한
                    게시글을 정말 삭제하시겠습니까?
                  </p>
                </b-modal>
              </div>
              <div v-else>
                <b-dropdown-item href="#" variant="danger" @click="reportPost"
                  >신고</b-dropdown-item
                >
              </div>
            </b-dropdown>
          </b-col>
        </b-row>
      </b-card-text>

      <!-- 2. 중앙 부분 -->
      <!--2.1 이미지-->
      <b-row v-if="fileId.length > 0" align-h="center">
        <b-carousel
          id="carousel-1"
          v-if="fileId.length > 0"
          v-model="slide"
          controls
          indicators
          background="#ababab"
          img-width="1024"
          img-height="480"
          style="text-shadow: 1px 1px 2px #333; width: 100%; height: 20rem;"
          :fade="true"
          :interval="0"
        >
          <b-carousel-slide
            id="post_img"
            v-for="(item, index) in fileId"
            :key="index"
            :img-src="url + `/userpost/download/` + item"
          ></b-carousel-slide>
        </b-carousel>
      </b-row>
      <!--2.2 내용-->
      <b-row class="mt-3 mx-5" align-h="center">
        <div class="my-3 mx-3" style="text-align: left;">
          <h6 @click="detail(post)" v-html="post.postContent"></h6>
        </div>
      </b-row>

      <b-row class="h2 mb-2 ml-2" align-h="start">
        <!-- 좋아요 -->
        <div class="postLike mr-2" style="margin-top: 2px;">
          <div class="h4" v-if="liked">
            <b-icon icon="suit-heart-fill" variant="danger" @click="likePost()"></b-icon>
          </div>
          <div class="h4" v-else>
            <b-icon icon="suit-heart" variant="danger" @click="likePost()"></b-icon>
          </div>
        </div>
        <!-- 댓글 수 -->
        <div class="postComment" @click="showComment">
          <div class="h4" v-if="commentFlag">
            <b-icon icon="chat-fill" variant="warning"></b-icon>
            <span class="ml-1" style="color:orange"
              ><small>{{ commentCount }}</small></span
            >
          </div>
          <div class="h4" v-else>
            <b-icon icon="chat" variant="warning"></b-icon>
            <span class="ml-1" style="color:orange"
              ><small>{{ commentCount }}</small></span
            >
          </div>
        </div>
      </b-row>

      <b-row class="ml-2 mb-3">
        <div v-if="post.postLikeCount >= 0">{{ post.postLikeCount }}명이 좋아합니다</div>
      </b-row>

      <!-- 댓글 -->
      <div v-show="commentFlag">
        <div v-if="comments.length > 0" style="width: 80%; display: inline-block">
          <div v-for="(comm, i) in comments" :key="i">
            <Comment :comment="comm" type="userpost" />
          </div>
        </div>
        <div v-else>
          아직 댓글이 없어요! 처음으로 댓글을 달아보세요!
        </div>

        <b-row class="mt-3" v-if="comments.length > 0 && comments.length < commentCount">
          <b-col>
            <span style="cursor: pointer;" @click="getMoreComments">
              <!-- <b-button pill variant="light" @click="getMoreComments">+</b-button> -->
              <img alt="Vue logo" src="@/assets/udonge.png" style="width: 5%;" />더보기
            </span>
          </b-col>
        </b-row>
      </div>
      <br />
      <!--댓글 입력창-->
      <div class="container">
        <b-row align-h="justify">
          <b-col cols="8" offset="1"
            ><b-form-input
              placeholder="댓글을 달아보세요!"
              v-model="comment"
              @keypress.enter="writeComment"
            ></b-form-input
          ></b-col>
          <b-col cols="2"
            ><b-button type="submit" variant="info" @click="writeComment">확인</b-button></b-col
          >
        </b-row>
      </div>
    </b-card>
  </div>
</template>

<script>
//import ImageSlick from '@/components/story/ImageSlick'
import Comment from '@/components/story/Comment';
import axios from 'axios';

const SERVER_URL = process.env.VUE_APP_SERVER_URL;

export default {
  name: 'PostBlockMy',
  components: {
    Comment,
  },
  props: {
    post: Object,
  },
  data() {
    return {
      postDetail: Object,
      fileId: Object,
      url: SERVER_URL,
      userId: '',
      liked: false,
      comments: [], //글에 달린 댓글
      commentCount: 0,
      commentFlag: false,
      limit: 5,
      offset: 0,
      comment: '', //작성하는 댓글
      badge: '',
    };
  },
  created() {
    const userInfo = JSON.parse(localStorage.getItem('Info-token'));
    this.userId = userInfo['userId'];
    const userInfo2 = JSON.parse(localStorage.getItem('Login-token'));
    this.badge = userInfo2.user_badge;

    axios.get(`${SERVER_URL}/userpost/${this.post.postId}`).then((res) => {
      this.postDetail = res;
      this.fileId = res.data.fileId;
    });

    this.getLikeInfo(); //해당 게시글에 좋아요를 눌렀는지 확인
    this.getComments(); //게시글에 달린 댓글 가져오기
  },
  methods: {
    toFeed: function() {
      // 자신의 배지를 누르기 때문에 새로고침! 기록을 남기지 않기 위해 replace 사용
      this.$router.push({
        name: 'MyFeed',
        params: { userId: this.post.userId, nickname: this.post.nickname },
      });
    },
    getLikeInfo() {
      axios
        .get(`${SERVER_URL}/userpost/like`, {
          params: {
            userId: this.userId,
            postId: this.post.postId,
          },
        })
        .then((response) => (this.liked = response.data));
    },
    deletePost() {
      axios.delete(`${SERVER_URL}/userpost?postId=${this.post.postId}`).then((response) => {
        console.log(response);
        location.reload(true);
      });
    },
    reportPost() {
      alert("신고되었습니다.")
    },
    likePost() {
      axios
        .post(`${SERVER_URL}/userpost/like`, {
          postId: this.post['postId'],
          userId: this.userId,
        })
        .then((response) => {
          this.liked = !response.data.includes('취소');
          if (this.liked) {
            this.post['postLikeCount'] = this.post['postLikeCount'] * 1 + 1;
          } else {
            this.post['postLikeCount'] = this.post['postLikeCount'] * 1 - 1;
          }
        });
    },
    showComment() {
      this.commentFlag = !this.commentFlag;
    },
    getComments() {
      axios
        .get(`${SERVER_URL}/userpost/comment`, {
          params: {
            postId: this.post.postId,
            limit: this.limit,
            offset: this.offset,
          },
        })
        .then((response) => {
          this.comments.push(...response.data.list);
          this.commentCount = response.data.count;
        });
    },
    getMoreComments() {
      if (this.commentCount <= this.comments.length) {
        return;
      }

      this.offset += this.limit;
      this.getComments();
    },
    writeComment() {
      axios
        .post(`${SERVER_URL}/userpost/comment`, {
          postId: this.post['postId'],
          userId: this.userId,
          commContent: this.comment,
        })
        .then((response) => {
          console.log(response);
          this.offset = 0;
          this.comments = []; //댓글 초기화
          this.getComments();
          this.post.postCommentCount = this.post.postCommentCount * 1 + 1;
          this.comment = '';
          if (this.commentFlag === false) {
            this.showComment();
          }
        });
    },
    showModal() {
      this.$refs[`${this.post.postId}`].show();
    },
  },
};
</script>

<style>
#post_img {
  top: 0%;
  left: 0%;
  min-width: 100%;
  min-height: 20em;
  max-width: 100%;
  max-height: 20em;
}
/* 적용안됨 */
b-card-img {
  height: 5px;
  width: 5px;
}
</style>
