<template>
  <div class="ar-patient-ui">
    <div class="header">
      <h1>光学体表监测系统</h1>
      <div class="patient-info">
        <p>患者ID: 123-456-789</p>
        <p>扫描时间: {{ currentDateTime }}</p>
      </div>
    </div>
    
    <div class="main-content">
      <div class="body-scan">
        <h2>体表深度图成像</h2>
        <div class="scan-container">
          <img src="@/assets/humanbody-depth.gif" alt="3D Body Scan" class="scan-image" />
        </div>
      </div>
      
      <div class="vital-signs">
        <div class="vital-sign">
          <h2>心率</h2>
          <div class="vital-data">
            <div class="vital-value">{{ pulseRate }} <span class="unit">BPM</span></div>
            <div class="waveform-container">
              <canvas ref="pulseCanvas" class="waveform"></canvas>
            </div>
          </div>
        </div>
        
        <div class="vital-sign">
          <h2>呼吸率</h2>
          <div class="vital-data">
            <div class="vital-value">{{ breathRate }} <span class="unit">次/分</span></div>
            <div class="waveform-container">
              <canvas ref="breathCanvas" class="waveform"></canvas>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="footer">
      <p>© AR界面演示</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ARPatientUI',
  data() {
    return {
      pulseRate: 72,
      breathRate: 16,
      currentDateTime: '',
      pulseWaveform: null,
      breathWaveform: null,
      animationFrameId: null,
      dataPointsBuffer: 5
    }
  },
  mounted() {
    this.updateDateTime();
    setInterval(this.updateDateTime, 1000);
    this.initWaveforms();
    this.startSimulation();
    
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    cancelAnimationFrame(this.animationFrameId);
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    updateDateTime() {
      const now = new Date();
      this.currentDateTime = now.toLocaleString('zh-CN');
    },
    handleResize() {
      this.initWaveforms();
    },
    initWaveforms() {
      const pulseCanvas = this.$refs.pulseCanvas;
      const pulseCtx = pulseCanvas.getContext('2d');
      pulseCanvas.width = pulseCanvas.parentElement.clientWidth;
      pulseCanvas.height = pulseCanvas.parentElement.clientHeight;
      
      const breathCanvas = this.$refs.breathCanvas;
      const breathCtx = breathCanvas.getContext('2d');
      breathCanvas.width = breathCanvas.parentElement.clientWidth;
      breathCanvas.height = breathCanvas.parentElement.clientHeight;
      
      this.pulseWaveform = {
        canvas: pulseCanvas,
        ctx: pulseCtx,
        data: Array(pulseCanvas.width).fill(pulseCanvas.height / 2),
        color: '#ff4d4d',  // Red for heart rate
        counter: 0
      };
      
      this.breathWaveform = {
        canvas: breathCanvas,
        ctx: breathCtx,
        data: Array(breathCanvas.width).fill(breathCanvas.height / 2),
        color: '#4dff88',  // Green for breath rate
        counter: 0
      };
    },
    startSimulation() {
      const animate = () => {
        this.updatePulseWaveform();
        this.updateBreathWaveform();
        this.updateVitalSigns();
        this.animationFrameId = requestAnimationFrame(animate);
      };
      
      animate();
    },
    updatePulseWaveform() {
      const { canvas, ctx, data, color, counter } = this.pulseWaveform;
      
      this.pulseWaveform.counter = (counter + 1) % this.dataPointsBuffer;
      if (this.pulseWaveform.counter !== 0) return;
      
      data.shift();
      
      const time = Date.now() / 1000;
      const pulseValue = Math.sin(time * 4) * 30 + 
                         Math.sin(time * 8) * 18 + 
                         Math.random() * 1.5;
      data.push(canvas.height / 2 - pulseValue);
      
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      
      this.drawGrid(ctx, canvas.width, canvas.height);
      
      ctx.beginPath();
      ctx.moveTo(0, data[0]);
      for (let i = 1; i < data.length; i++) {
        ctx.lineTo(i, data[i]);
      }
      ctx.strokeStyle = color;
      ctx.lineWidth = 3.5;
      ctx.stroke();
    },
    updateBreathWaveform() {
      const { canvas, ctx, data, color, counter } = this.breathWaveform;
      
      this.breathWaveform.counter = (counter + 1) % this.dataPointsBuffer;
      if (this.breathWaveform.counter !== 0) return;
      
      data.shift();
      
      const time = Date.now() / 1000;
      const breathValue = Math.sin(time * 1) * 50 + Math.random() * 1.5;
      data.push(canvas.height / 2 - breathValue);
      
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      
      this.drawGrid(ctx, canvas.width, canvas.height);
      
      ctx.beginPath();
      ctx.moveTo(0, data[0]);
      for (let i = 1; i < data.length; i++) {
        ctx.lineTo(i, data[i]);
      }
      ctx.strokeStyle = color;
      ctx.lineWidth = 3.5;
      ctx.stroke();
    },
    drawGrid(ctx, width, height) {
      ctx.beginPath();
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.15)';
      ctx.lineWidth = 1;
      
      for (let y = 0; y <= height; y += 20) {
        ctx.moveTo(0, y);
        ctx.lineTo(width, y);
      }
      
      for (let x = 0; x <= width; x += 50) {
        ctx.moveTo(x, 0);
        ctx.lineTo(x, height);
      }
      
      ctx.stroke();
    },
    updateVitalSigns() {
      const time = Date.now() / 1000;
      this.pulseRate = Math.round(72 + Math.sin(time / 10) * 3);
      this.breathRate = Math.round(16 + Math.sin(time / 15) * 2);
    }
  }
}
</script>

<style scoped>
.ar-patient-ui {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  color: #ffffff;
  background-color: rgba(11, 70, 88, 0.95);
  padding: 15px 20px;
  border-radius: 10px;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  aspect-ratio: 16/10;
  max-height: 100vh;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  padding-bottom: 8px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.3);
  height: 8%;
}

.header h1 {
  font-size: 22px;
  margin: 0;
  color: #ffffff;
  font-weight: 500;
}

.patient-info {
  text-align: right;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.9);
}

.patient-info p {
  margin: 3px 0;
}

.main-content {
  display: flex;
  flex: 1;
  gap: 15px;
  flex-direction: row;
  height: 85%;
}

.body-scan {
  flex: 0 0 55%;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 12px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
}

.body-scan h2 {
  color: #ffffff;
  font-size: 18px;
  margin-top: 0;
  margin-bottom: 8px;
  height: 24px;
}

.scan-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.scan-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.vital-signs {
  flex: 0 0 43%;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.vital-sign {
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 12px;
  flex: 1;
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
}

.vital-sign h2 {
  font-size: 18px;
  margin-top: 8px;
  margin-left: 20px;
  margin-bottom: 10px;
  color: #ffffff;
  font-weight: 500;
  height: 24px;
}

.vital-data {
  display: flex;
  align-items: center;
  flex: 1;
  gap: 15px;
}

.vital-value {
  font-size: 32px;
  font-weight: bold;
  min-width: 90px;
  text-align: center;
  color: #ffffff;
}

.unit {
  font-size: 14px;
  font-weight: normal;
  opacity: 0.8;
}

.waveform-container {
  flex: 1;
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 4px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.15);
  height: 80%;
  min-height: 80px; /* Ensure minimum height for waveforms */
}

.waveform {
  width: 100%;
  height: 100%;
}

.footer {
  margin-top: 10px;
  text-align: center;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.7);
  height: 5%;
}

/* Ensure proper aspect ratio for different screen sizes */
@media (max-width: 1600px) {
  .body-scan {
    flex: 0 0 53%;
  }
  
  .vital-signs {
    flex: 0 0 45%;
  }
}

@media (max-width: 1200px) {
  .body-scan {
    flex: 0 0 50%;
  }
  
  .vital-signs {
    flex: 0 0 48%;
  }
}

@media (max-width: 1024px) {
  .ar-patient-ui {
    aspect-ratio: auto;
  }
  
  .main-content {
    flex-direction: column;
  }
  
  .body-scan, .vital-signs {
    flex: none;
    width: 100%;
  }
  
  .body-scan {
    height: 60%;
  }
  
  .vital-signs {
    height: 40%;
  }
}
</style> 