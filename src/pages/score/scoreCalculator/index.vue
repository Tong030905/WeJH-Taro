<template>
  <theme-config>
    <title-bar title="绩点计算器" back-button />
    <scroll-view :scrollY="true">
      <view class="flex-column">
        <card v-if="!scoreList || scoreList.length === 0" style="text-align: center">
          <view>无当前阶段成绩信息</view>
        </card>

        <card v-else class="score-card">
          <template #header>
            <view class="row">
              <view class="score-icon-wrapper">
                <view class="score-icon iconfont icon-score" />
              </view>
              <view class="col">
                <view class="term-info">{{ termInfo }}</view>
                <view class="relative-term-info">{{ relativeTermInfo }}</view>
              </view>
            </view>

            <view class="col">
              <view class="gpa-text">
                {{ calTerm.period === "期中" ? "期中" : "GPA" }}
              </view>
              <view v-if="scoreList && scoreList.length !== 0 && calTerm.period === '期末'" class="credit-text">
                {{ averageScorePoint }}
              </view>
            </view>

            <view v-if="!isEdit" class="col" style="align-items: flex-end" @tap="handleSwitch()">
                <view class="iconfont icon-setting switch-btn"></view>
            </view>

            <view v-if="isEdit" class="col" style="align-items: flex-end" @tap="handleSwitch()">
                <view class="iconfont icon-save-calculation switch-btn"></view>
            </view>
          </template>

          <w-collapse class="score-list-collapse" v-if="!isEdit">
            <w-collapse-panel v-for="item in scoreList" :key="item.lessonID" arrow>
              <template #header>
                <checkbox-group @change="handleCheckboxChange(item)" v-if="isEdit">
                  <checkbox class="checkbox" :checked="item.selected"
                  ></checkbox>
                </checkbox-group>
                <view class="score-list-collapse-item-title">
                  {{ item.lessonName }}
                </view>
                <view class="score-list-collapse-item-extra">
                  {{ item.score }}
                </view>
              </template>

              <w-descriptions class="score-detail-list" size="small">
                <w-descriptions-item label="课程名称">
                  {{ item.lessonName }}
                </w-descriptions-item>
                <w-descriptions-item v-if="item.lessonType" label="课程性质">
                  {{ item.lessonType }}
                </w-descriptions-item>
                <w-descriptions-item label="课程学分">
                  {{ item.credits }}
                </w-descriptions-item>
              </w-descriptions>
            </w-collapse-panel>
          </w-collapse>


          <w-button class="lesson-group-btn" v-if="isEdit" @tap="requireLessonChange()">必修课</w-button>
          <w-collapse class="score-list-collapse" v-if="isEdit">
            <w-collapse-panel v-for="item in requiredScoreList" :key="item.lessonID">
              <template #header>
                <checkbox-group @change="handleCheckboxChange(item)" v-if="isEdit">
                  <checkbox class="checkbox" :checked="item.selected"
                  ></checkbox>
                </checkbox-group>
                <view class="score-list-collapse-item-title">
                  {{ item.lessonName }}
                </view>
                <view class="score-list-collapse-item-extra">
                  {{ item.score }}
                </view>
              </template>
            </w-collapse-panel>
          </w-collapse>

          <w-button class="lesson-group-btn" v-if="isEdit" @tap="optionalLessonChange()">任选课</w-button>
          <w-collapse class="score-list-collapse" v-if="isEdit">
            <w-collapse-panel v-for="item in optionalScoreList" :key="item.lessonID">
              <template #header>
                <checkbox-group @change="handleCheckboxChange(item)" v-if="isEdit">
                  <checkbox class="checkbox" :checked="item.selected"
                  ></checkbox>
                </checkbox-group>
                <view class="score-list-collapse-item-title">
                  {{ item.lessonName }}
                </view>
                <view class="score-list-collapse-item-extra">
                  {{ item.score }}
                </view>
              </template>
            </w-collapse-panel>
          </w-collapse>

          <w-button class="lesson-group-btn" v-if="isEdit" @tap="sportsLessonChange()">体育课</w-button>
          <w-collapse class="score-list-collapse" v-if="isEdit">
            <w-collapse-panel v-for="item in sportsScoreList" :key="item.lessonID">
              <template #header>
                <checkbox-group @change="handleCheckboxChange(item)" v-if="isEdit">
                  <checkbox class="checkbox" :checked="item.selected"
                  ></checkbox>
                </checkbox-group>
                <view class="score-list-collapse-item-title">
                  {{ item.lessonName }}
                </view>
                <view class="score-list-collapse-item-extra">
                  {{ item.score }}
                </view>
              </template>
            </w-collapse-panel>
          </w-collapse>
        </card>
        <card v-if="scoreList?.length !== 0">
          <view class="score-help">{{ helpContent }}</view>
        </card>
      </view>
    </scroll-view>
  </theme-config>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import Taro from "@tarojs/taro";
import {
  Card,
  TitleBar,
  WCollapsePanel,
  WCollapse,
  WButton,
  WDescriptions,
  WDescriptionsItem,
  ThemeConfig
} from "@/components";
import { Score } from "@/types/Score";
import { ZFService } from "@/services";
import { helpText } from "@/constants/copywriting";
import store, { serviceStore } from "@/store";
import "./index.scss";

type Term = {
  year: string;
  term: string;
  period: "期中" | "期末";
};

const isEdit = ref(false);
const instance = Taro.getCurrentInstance();

const { term, year } = instance.router?.params as {
  term: string;
  year: string;
};

const calTerm = ref<Term>({
  year: year,
  term: term,
  period: "期末"
});

const scoreList = computed(() => {
  const data = ZFService.getScoreInfo(calTerm.value).data;
  data.forEach(item => {
    // existingScore存储不纳入计算的成绩
    const existingScore = serviceStore.score.unCalScore.find(
      storeItem => (item.lessonID === storeItem.name &&
      item.scorePoint === storeItem.scorePoint)
    );
    if (!existingScore) item.selected = true;
    else item.selected = false;
  });
  return data;
});

const requiredScoreList = computed(() => {
  return scoreList.value.filter(item => item.lessonType === "必修课");
});

const sportsScoreList = computed(() => {
  return scoreList.value.filter(item => item.lessonType === "体育课");
});

const optionalScoreList = computed(() => {
  return scoreList.value.filter(item => item.lessonType === "任选课");
});

const allChosen_1 = ref(false);
const allChosen_2 = ref(false);
const allChosen_3 = ref(false);

const requireLessonChange = () => {
  if (!allChosen_1.value) {
    requiredScoreList.value.forEach(item => {
      if (!item.selected) selectedLessons.value.push(item);
      item.selected = true;
      store.commit("delUnCalc", item);
    });
  }else{
    requiredScoreList.value.forEach(item => {
      if (item.selected) {
        selectedLessons.value = selectedLessons.value.filter(
          selected => selected.lessonID !== item.lessonID
        );
        store.commit("setUnCalc", item);
      }
      item.selected = false;
    });
  }
  allChosen_1.value = !allChosen_1.value;
};

const sportsLessonChange = () => {
  if (!allChosen_2.value) {
    sportsScoreList.value.forEach(item => {
      if (!item.selected) selectedLessons.value.push(item);
      item.selected = true;
      store.commit("delUnCalc", item);
    });
  }else{
    sportsScoreList.value.forEach(item => {
      if (item.selected) {
        selectedLessons.value = selectedLessons.value.filter(
          selected => selected.lessonID !== item.lessonID
        );
        store.commit("setUnCalc", item);
      }
      item.selected = false;
    });
  }
  allChosen_2.value = !allChosen_2.value;
};

const optionalLessonChange = () => {
  if (!allChosen_3.value) {
    optionalScoreList.value.forEach(item => {
      if (!item.selected) selectedLessons.value.push(item);
      item.selected = true;
      store.commit("delUnCalc", item);
    });
  }else{
    optionalScoreList.value.forEach(item => {
      if (item.selected) {
        selectedLessons.value = selectedLessons.value.filter(
          selected => selected.lessonID !== item.lessonID
        );
        store.commit("setUnCalc", item);
      }
      item.selected = false;
    });
  }
  allChosen_3.value = !allChosen_3.value;
};

const selectedLessons = ref(scoreList.value.filter(item => item.selected));

function handleCheckboxChange(item) {
  item.selected = !item.selected;
  if (item.selected) {
    selectedLessons.value.push(item);
    store.commit("delUnCalc", item);
  } else {
    //将这个课程从selectedLessons中删除
    selectedLessons.value = selectedLessons.value.filter(
      selected => selected.lessonID !== item.lessonID
    );
    store.commit("setUnCalc", item);
  }
}

const isRefreshing = ref(false);

const helpContent = computed(() => {
  return helpText.score;
});


async function refresh() {
  if (isRefreshing.value) return;
  isRefreshing.value = true;
  await ZFService.updateScoreInfo(calTerm.value);
  isRefreshing.value = false;
}

onMounted(async () => {
  if (serviceStore.user.isBindZF) await refresh();
});

const averageScorePoint = computed(() => {
  const validCourse = selectedLessons.value.filter((item) => {
    if (item.score === "缓考" || item.score === "免修") return false;
    if (item.examType === "重修" || item.examType === "补考")
      return false;
    return true;
  });
  let totalCredits = 0;
  let totalScorePoint = 0;
  validCourse.forEach((item: Score) => {
    const scorePoint = parseFloat(item.scorePoint);
    const credits = parseFloat(item.credits);
    totalScorePoint += scorePoint * credits;
    totalCredits += credits;
  });
  const ans = Math.floor((totalScorePoint / totalCredits) * 1000) / 1000;
  if (ans !== ans) return "";
  else return ans;
});

const termInfo = computed(() => {
  return `
    ${calTerm.value?.year}/${parseInt(calTerm.value?.year) + 1}
    （${calTerm.value?.term}）
  `;
});

const relativeTermInfo = computed(() => {
  const charEnum = ["一", "二", "三", "四", "五", "六", "日"];
  let char = charEnum[0];
  if (serviceStore.user.info?.studentID) {
    char = charEnum[
      parseInt(calTerm.value?.year) -
      parseInt(serviceStore.user.info.studentID.slice(0, 4))
    ];
  }
  return `大${char}${calTerm.value?.term}学期`;
});

const handleSwitch = () => {
  isEdit.value = !isEdit.value;
};

</script>
