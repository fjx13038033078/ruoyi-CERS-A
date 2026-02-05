<template>
  <div class="china-map-container">
    <!-- 页面标题 -->
    <div class="page-header">
      <div class="header-content">
        <div class="header-icon">
          <i class="el-icon-map-location"></i>
        </div>
        <div class="header-text">
          <h1>全国高校分布地图</h1>
          <p>可视化展示中国各省份高校分布情况，点击省份查看详细信息</p>
        </div>
      </div>
    </div>

    <!-- 主内容区域 -->
    <el-row :gutter="20" class="main-content">
      <!-- 左侧统计信息 -->
      <el-col :span="6">
        <div class="stat-panel">
          <div class="stat-title">
            <i class="el-icon-data-analysis"></i>
            <span>数据统计</span>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon blue">
              <i class="el-icon-school"></i>
            </div>
            <div class="stat-info">
              <div class="stat-value">{{ totalUniversities }}</div>
              <div class="stat-label">全国高校总数</div>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon red">
              <i class="el-icon-trophy"></i>
            </div>
            <div class="stat-info">
              <div class="stat-value">{{ total985 }}</div>
              <div class="stat-label">985高校数量</div>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon green">
              <i class="el-icon-medal"></i>
            </div>
            <div class="stat-info">
              <div class="stat-value">{{ total211 }}</div>
              <div class="stat-label">211高校数量</div>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon orange">
              <i class="el-icon-location-outline"></i>
            </div>
            <div class="stat-info">
              <div class="stat-value">{{ totalProvinces }}</div>
              <div class="stat-label">覆盖省份数</div>
            </div>
          </div>

          <div class="tips-panel">
            <div class="tips-title">
              <i class="el-icon-info"></i>
              <span>使用提示</span>
            </div>
            <ul class="tips-list">
              <li>鼠标悬停省份可查看详细数据</li>
              <li>颜色深浅表示高校数量多少</li>
              <li>拖动左下角滑块可筛选数量范围</li>
            </ul>
          </div>
        </div>
      </el-col>

      <!-- 右侧地图区域 -->
      <el-col :span="18">
        <div class="map-panel">
          <div id="china-map" class="map-container"></div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import {MapChart} from "echarts/charts";
import {TitleComponent, TooltipComponent, VisualMapComponent} from "echarts/components";
import {CanvasRenderer} from "echarts/renderers";
import chinaMap from "@/assets/geo/china.json";
import * as echarts from "echarts";
import {listProvinces} from "@/api/university/province";

// 注册必要组件
echarts.use([MapChart, TooltipComponent, VisualMapComponent, TitleComponent, CanvasRenderer]);

export default {
  data() {
    return {
      mapData: [], // 存放高校数据
      chartInstance: null, // 存储 ECharts 实例
    };
  },
  computed: {
    // 计算统计数据
    totalUniversities() {
      return this.mapData.reduce((sum, item) => sum + (item.value || 0), 0);
    },
    total985() {
      return this.mapData.reduce((sum, item) => sum + (item.num985 || 0), 0);
    },
    total211() {
      return this.mapData.reduce((sum, item) => sum + (item.num211 || 0), 0);
    },
    totalProvinces() {
      return this.mapData.filter(item => item.value > 0).length;
    }
  },
  mounted() {
    this.fetchProvincesData(); // 页面加载时调用接口
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize);
    if (this.chartInstance) {
      this.chartInstance.dispose();
    }
  },
  methods: {
    handleResize() {
      if (this.chartInstance) {
        this.chartInstance.resize();
      }
    },
    /** 获取省份高校数据 */
    async fetchProvincesData() {
      try {
        const response = await listProvinces({ pageSize: 100});
        if (response.code === 200) {
          this.mapData = response.rows.map((item) => ({
            name: item.provinceName, // 省份名称
            value: item.numTotal, // 高校总数
            num985: item.num985, // 985高校数
            num211: item.num211, // 211高校数
            minScoreLiberal2024: item.minScoreLiberal2024,
            minScoreScience2024: item.minScoreScience2024,
            minScoreLiberal2023: item.minScoreLiberal2023,
            minScoreScience2023: item.minScoreScience2023,
            minScoreLiberal2022: item.minScoreLiberal2022,
            minScoreScience2022: item.minScoreScience2022,
          }));
          this.initChinaMap(); // 数据加载后初始化地图
        } else {
          console.error("数据加载失败:", response.msg);
        }
      } catch (error) {
        console.error("请求失败:", error);
      }
    },
    /** 初始化中国地图 */
    initChinaMap() {
      if (!this.chartInstance) {
        this.chartInstance = echarts.init(this.$el.querySelector("#china-map"));
      }

      // 注册中国地图
      echarts.registerMap("china", chinaMap);

      // 配置项
      const option = {
        tooltip: {
          trigger: "item",
          backgroundColor: 'rgba(255, 255, 255, 0.95)',
          borderColor: '#e4e7ed',
          borderWidth: 1,
          padding: [12, 16],
          textStyle: {
            color: '#303133',
            fontSize: 13
          },
          formatter: (params) => {
            const {name, data} = params;
            if (!data) {
              return `<div style="font-weight:600;font-size:15px;margin-bottom:8px;">${name}</div><div style="color:#909399;">暂无数据</div>`;
            }
            const value = data.value || 0;
            const num985 = data.num985 || 0;
            const num211 = data.num211 || 0;
            const minScoreLiberal2024 = data.minScoreLiberal2024 || '-';
            const minScoreScience2024 = data.minScoreScience2024 || '-';
            return `
              <div style="font-weight:600;font-size:15px;margin-bottom:10px;color:#1a5fb4;">${name}</div>
              <div style="display:flex;justify-content:space-between;margin-bottom:6px;">
                <span style="color:#606266;">高校总数</span>
                <span style="font-weight:600;color:#303133;">${value} 所</span>
              </div>
              <div style="display:flex;justify-content:space-between;margin-bottom:6px;">
                <span style="color:#606266;">985高校</span>
                <span style="font-weight:600;color:#c01c28;">${num985} 所</span>
              </div>
              <div style="display:flex;justify-content:space-between;margin-bottom:10px;">
                <span style="color:#606266;">211高校</span>
                <span style="font-weight:600;color:#26a269;">${num211} 所</span>
              </div>
              <div style="border-top:1px solid #ebeef5;padding-top:10px;font-size:12px;">
                <div style="margin-bottom:4px;color:#909399;">2024最低投档线</div>
                <div style="display:flex;justify-content:space-between;">
                  <span>历史类: <b>${minScoreLiberal2024}</b></span>
                  <span>物理类: <b>${minScoreScience2024}</b></span>
                </div>
              </div>
            `;
          },
        },
        visualMap: {
          min: 0,
          max: 180,
          left: 20,
          bottom: 20,
          text: ["高校数量多", "高校数量少"],
          textStyle: {
            color: '#606266',
            fontSize: 12
          },
          inRange: {
            color: ["#e8f4ff", "#b3d4fc", "#7eb8f7", "#4a9cf2", "#1a5fb4"],
          },
          calculable: true,
        },
        series: [
          {
            name: "高校分布",
            type: "map",
            map: "china",
            roam: true,
            zoom: 1.2,
            scaleLimit: {
              min: 0.8,
              max: 3
            },
            label: {
              show: true,
              fontSize: 10,
              color: '#606266'
            },
            itemStyle: {
              areaColor: '#f5f7fa',
              borderColor: '#c0c4cc',
              borderWidth: 1
            },
            emphasis: {
              label: {
                show: true,
                fontSize: 12,
                fontWeight: 'bold',
                color: '#1a5fb4'
              },
              itemStyle: {
                areaColor: '#ffd666',
                borderColor: '#1a5fb4',
                borderWidth: 2
              }
            },
            select: {
              label: {
                show: true,
                color: '#1a5fb4'
              },
              itemStyle: {
                areaColor: '#ffe58f'
              }
            },
            data: this.mapData,
          },
        ],
      };
      // 设置选项
      this.chartInstance.setOption(option);
    },
  }
}
</script>


<style scoped lang="scss">
.china-map-container {
  padding: 20px;
  background: #f5f7fa;
  min-height: calc(100vh - 84px);
}

.page-header {
  background: #fff;
  border: 1px solid #e4e7ed;
  padding: 24px;
  margin-bottom: 20px;
  
  .header-content {
    display: flex;
    align-items: center;
    
    .header-icon {
      width: 56px;
      height: 56px;
      background: #1a5fb4;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-right: 20px;
      
      i {
        font-size: 28px;
        color: #fff;
      }
    }
    
    .header-text {
      h1 {
        margin: 0 0 8px 0;
        font-size: 22px;
        color: #303133;
        font-weight: 600;
      }
      
      p {
        margin: 0;
        font-size: 14px;
        color: #909399;
      }
    }
  }
}

.main-content {
  .stat-panel {
    background: #fff;
    border: 1px solid #e4e7ed;
    padding: 20px;
    height: 100%;
    
    .stat-title {
      display: flex;
      align-items: center;
      font-size: 16px;
      font-weight: 600;
      color: #303133;
      margin-bottom: 20px;
      padding-bottom: 12px;
      border-bottom: 1px solid #ebeef5;
      
      i {
        margin-right: 8px;
        color: #1a5fb4;
        font-size: 18px;
      }
    }
    
    .stat-card {
      display: flex;
      align-items: center;
      padding: 16px;
      background: #f5f7fa;
      border: 1px solid #e4e7ed;
      margin-bottom: 12px;
      transition: all 0.2s ease;
      
      &:hover {
        background: #e8f4ff;
        border-color: #1a5fb4;
      }
      
      .stat-icon {
        width: 44px;
        height: 44px;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-right: 14px;
        
        i {
          font-size: 22px;
          color: #fff;
        }
        
        &.blue { background: #1a5fb4; }
        &.red { background: #c01c28; }
        &.green { background: #26a269; }
        &.orange { background: #e5a50a; }
      }
      
      .stat-info {
        .stat-value {
          font-size: 24px;
          font-weight: 700;
          color: #303133;
          line-height: 1.2;
        }
        
        .stat-label {
          font-size: 13px;
          color: #909399;
          margin-top: 4px;
        }
      }
    }
    
    .tips-panel {
      margin-top: 20px;
      padding: 16px;
      background: #fef0e6;
      border: 1px solid #f5d0b5;
      
      .tips-title {
        display: flex;
        align-items: center;
        font-size: 14px;
        font-weight: 600;
        color: #e5a50a;
        margin-bottom: 12px;
        
        i {
          margin-right: 6px;
        }
      }
      
      .tips-list {
        margin: 0;
        padding-left: 18px;
        
        li {
          font-size: 13px;
          color: #606266;
          line-height: 1.8;
        }
      }
    }
  }
  
  .map-panel {
    background: #fff;
    border: 1px solid #e4e7ed;
    padding: 20px;
    height: 100%;
    
    .map-container {
      width: 100%;
      height: 700px;
    }
  }
}
</style>
