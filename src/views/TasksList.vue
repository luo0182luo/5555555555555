<template>
  <div class="tasks_list">
    <van-search
      class="tasks_list-search"
      v-model="searchVal"
      clearable
      :placeholder="$t('TasksList.searchPlaceholder')"
      @change="searchFn"
    />
    <van-pull-refresh v-model="refreshing" @refresh="searchFn">
      <van-cell icon="label" :title="title">
        <template #extra>
          <van-button
            icon="plus"
            size="mini"
            type="info"
            @click="add"
            :disabled="result.length==0"
          >{{$t('TasksList.add')}}</van-button>
        </template>
      </van-cell>
      <div class="man_hour-item-content">
        <van-checkbox-group v-model="result">
          <van-cell v-for="item in list" :key="item.id">
            <template>
              <div class="tasks_list-custom">
                <div class="tasks_list-custom-title">{{item.itemCode}}</div>
                <div class="tasks_list-custom-content">
                  <div>{{item.itemCont}}</div>
                  <div
                    v-if="item.fdProjectStage"
                    class="tasks_list-custom-content-text van-ellipsis"
                  >{{item.fdProjectStage}}</div>
                </div>
                <div class="tasks_list-custom-operate">
                  <van-checkbox
                    class="tasks_list-custom-operate-checkbox"
                    :name="item.id"
                    shape="square"
                  ></van-checkbox>
                </div>
              </div>
            </template>
          </van-cell>
        </van-checkbox-group>
      </div>
      <div>{{testText}}</div>
    </van-pull-refresh>
  </div>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  name: "TasksList",
  data() {
    return {
      searchVal: "",
      list: [],
      result: [],
      refreshing: false,
      testText: "",
    };
  },
  computed: {
    type() {
      return this.$route.params.type;
    },
    title() {
      return (
        this.$route.params.type &&
        this.$t(`TasksList.${this.$route.params.type}`)
      );
    },
  },
  methods: {
    add() {
      this.addTask({
        type: this.type,
        list: this.list.filter((item) =>
          this.result.some((resultItem) => resultItem === item.id)
        ),
      });
      this.$router.go(-1);
    },
    searchFn() {
      switch (this.type) {
        case "items":
          this.$axios
            .post("/api/worktime/dTalk/selectProjectInfoById", {
              name: this.searchVal,
              userId: this.$userId,
            })
            .then((res) => {
              this.testText = JSON.stringify(res);
              this.list = this.list = res.data.map((item) => {
                return {
                  id: item.id,
                  itemCode: item.projectCode,
                  itemCont: item.projectName,
                  fdProjectStage: item.projectStage,
                  fdProjectId: item.id,
                };
              });
            })
            .catch((err) => {
              this.testText = JSON.stringify(err);
            })
            .finally(() => {
              this.refreshing = false;
            });
          break;
        case "tasks":
          this.$axios
            .post("/api/worktime/dTalk/selectTaskInfoById", {
              name: this.searchVal,
              userId: this.$userId,
            })
            .then((res) => {
              this.testText = JSON.stringify(res);

              this.list = res.data.map((item) => {
                return {
                  id: item.id,
                  itemCode: item.taskCode,
                  itemCont: item.taskName,
                  fdTaskId: item.id,
                };
              });
            })
            .catch((err) => {
              this.testText = JSON.stringify(err);
            })
            .finally(() => {
              this.refreshing = false;
            });
          break;
        default:
          break;
      }
    },
    ...mapMutations(["addTask"]),
  },
  activated() {
    this.searchVal = "";
    this.result = [];
    this.searchFn();
  },
};
</script>

<style lang="less" scoped>
.tasks_list {
  /deep/ .van-icon {
    line-height: 24px;
  }
  .tasks_list-search {
    margin-bottom: 5px;
  }
  .tasks_list-custom {
    display: flex;
    justify-content: center;
    align-items: center;
    .tasks_list-custom-title {
      width: 90px;
    }
    .tasks_list-custom-content {
      flex: 1;
      overflow: hidden;
      justify-content: center;
      align-items: center;
      .tasks_list-custom-content-text {
        margin-top: 2px;
        color: #969799;
        font-size: 14px;
        line-height: 16px;
      }
    }
    .tasks_list-custom-operate {
      width: 30px;
      box-sizing: border-box;
      text-align: center;
      .tasks_list-custom-operate-add {
        font-size: 40px;
        vertical-align: middle;
        font-weight: bold;
      }
    }
  }
}
</style>
