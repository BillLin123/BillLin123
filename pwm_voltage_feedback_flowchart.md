graph TD
    %% PWM 週期與頻率設定
    subgraph PWM_Period [1. PWM 週期與頻率設定]
        direction TB
        A1([PWM_PeriodActuation<br>更新 PWM 頻率]) --> A2[設定 TIMA_CMP3 觸發 ADCy<br>hPWMPeriodHRTimTick/2 + Offset]
        A2 --> A3[設定 MASTER 週期<br>= hPWMPeriodHRTimTick]
        A3 --> A4[設定 PWM_FB_HS1_LS1 週期]
        A4 --> A5[設定 PWM_FB_HS2_LS2 週期]
    end

    %% 同步整流 (SR)
    subgraph SR_Control [2. 同步整流 (SR) 控制]
        direction TB
        B1([PWM_SynchRectActuation 相關函數<br>包含固定/可變死區時間更新]) --> B2[計算 SR 時序<br>Rising1/Falling1, Rising2/Falling2]
        B2 --> B3[邊緣驗證<br>限制最小值並處理越界 wrap-around]
        B3 --> B4{是否啟用自適應 SR<br>ADAPTIVE_SYNCH_RECTIFICATION ?}
        B4 -- 是 --> B5[設定消隱窗口 Blanking Window<br>並設定 ADC Vds 檢測觸發]
        B5 --> B6[更新 HRTIM 暫存器<br>設定 SR1/SR2 的 Period 與 Compare 值]
        B4 -- 否 --> B6
    end

    %% 死區時間控制
    subgraph Dead_Time [3. 死區時間 (Dead Time) 控制]
        direction TB
        C1([PWM_DeadTimeActuation<br>更新上升與下降邊緣死區時間]) --> C2[讀取當前 DTR 暫存器<br>HS1_LS1 & HS2_LS2]
        C2 --> C3[清除現有的死區時間數值]
        C3 --> C4[設定新的死區時間<br>DTR = hPWMDeadTimeDTTick]
        C4 --> C5[寫回 HRTIM DTxR 暫存器]
    end

    %% 打嗝模式 (Burst Mode)
    subgraph Burst_Mode [4. 打嗝模式 (Burst Mode) 控制]
        direction TB
        D1([PWM_UpdateBurstModeParams<br>配置 Burst 模式參數]) --> D2[設定 BMPER<br>總週期數 - 1]
        D2 --> D3[設定 BMCMPR<br>空閒週期數 - 1]
    end

    %% 風扇 PWM 控制
    subgraph Fan_Control [5. 風扇 PWM 控制]
        direction TB
        E1([PWM_FanActuation<br>更新風扇 PWM 佔空比]) --> E2[依據 Channel 寫入 TIM_CCR 暫存器<br>CCR1 / CCR2 / CCR3 / CCR4]
    end

    %% ADC 測量
    subgraph ADC_Measurements [6. ADC 測量轉換]
        direction TB
        F1([PWM_GetVdsSRMeasure<br>獲取 SR MOSFET Vds]) --> F2[從注入的 ADC 通道讀取<br>ADC1/ADC2 Injected Rank 1]
        
        F3([PWM_ConvertCurrentValue<br>安培轉換為 ADC 值]) --> F4[計算公式:<br>ADC = Current * Sensitivity / 3.3V * 4095 + Offset]
    end