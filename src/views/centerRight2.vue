<template>
  <div id="centerRight2">
    <div class="bg-color-black">
      <div class="d-flex pt-2 pl-2">
        <el-tooltip class="item" effect="dark" content="贡献者邮箱统计 数据来源: X-lab2017/open-digger" placement="top">
          <span class="text" style="font-size: 0.9rem; font-weight: bold;">贡献者邮箱</span>
        </el-tooltip>
      </div>
      <div class="d-flex ai-center flex-column body-box">
        <dv-capsule-chart class="dv-cap-chart" :config="config"  />
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import axios from 'axios'


export default {
  data() {
    return {
      config: {
        "data": [
          {
            "name": "gmail",
            "value": 3529
          },
          {
            "name": "qq",
            "value": 369
          },
          {
            "name": "github",
            "value": 351
          },
          {
            "name": "outlook",
            "value": 82
          },
          {
            "name": "163",
            "value": 76
          }
        ],
        "showValue": true
      }
    }
  },
  computed: {
    ...mapState(['currentRepository']),
  },
  watch: {
    currentRepository: {
      handler: async function (newVal) {
        this.config = await this.fetchData('https://oss.x-lab.info/open_digger/github/' + newVal);
        console.log(this.config);
      },
      deep: true
    }
  },
  methods: {
    async fetchData(path) {
      let mailResponse = await axios.get(path + '/contributor_email_suffixes.json');
      let mailData = mailResponse.data;

      return this.processData(mailData);
    },

    processData(data) {
      const result = {};
      Object.values(data).forEach(arr => {
        arr.forEach(item => {
          let [name, value] = item;

          if (name === 'users.noreply.github.com') {
            name = 'github.com';
          }

          const index = name.indexOf('.');
          if (index !== -1) {
            name = name.substring(0, index);
          }


          if (!result[name]) {
            result[name] = 0;
          }
          result[name] += Number(value);
        });
      });
      let output = Object.entries(result)
        .filter(([, value]) => value >= 75)
        .map(([name, value]) => ({ name, value }))
        .sort((a, b) => b.value - a.value);

      const configs = {
        data: output,
        showValue: true
      };

      return configs;
    }
  }
}
</script>

<style lang="scss" scoped>
#centerRight2 {
  $box-height: 410px;
  $box-width: 340px;
  padding: 5px;
  height: $box-height;
  width: $box-width;
  border-radius: 5px;

  .bg-color-black {
    padding: 5px;
    height: $box-height;
    width: $box-width;
    border-radius: 10px;
  }

  .text {
    color: #c3cbde;
  }

  .body-box {
    border-radius: 10px;
    overflow: hidden;

    .dv-cap-chart {
      width: 100%;
      height: 350px;
    }
  }
}
</style>
