graph TD
    %% Styling
    classDef hardware fill:#1a237e,stroke:#5c6bc0,stroke-width:2px,color:#fff
    classDef rfpath fill:#b71c1c,stroke:#ef5350,stroke-width:2px,color:#fff
    classDef baseband fill:#004d40,stroke:#26a69a,stroke-width:2px,color:#fff
    classDef dsp fill:#e65100,stroke:#ffa726,stroke-width:2px,color:#fff
    classDef env fill:#333333,stroke:#9e9e9e,stroke-width:2px,color:#fff

    %% 1. Frequency Synthesis
    subgraph SYNTH [1. Frequency Synthesis]
        CLK[100 MHz Ref Clock] ::: hardware
        PLL[DDS/PLL Synthesizer<br>24 GHz Staircase] ::: hardware
        CLK --> PLL
    end

    %% 2. RF Front-End & Environment
    subgraph RF [2. RF Front-End & Environment]
        PA[Power Amplifier] ::: rfpath
        CIRC((Circulator)) ::: rfpath
        ANT[TX/RX Antenna] ::: rfpath
        TGT((Moving Target<br>Echo Delay & Doppler)) ::: env
        LNA[Low Noise Amplifier<br>+ AWGN Noise] ::: rfpath

        PLL -- TX Signal --> PA
        PA --> CIRC
        CIRC <--> ANT
        ANT -. Free Space .- TGT
        CIRC -- RX Signal + Leakage --> LNA
        PA -. -20dB TX Leakage .-> LNA
    end

    %% 3. IQ Demodulation
    subgraph MIX [3. IQ Demodulator]
        SPLIT{90° Hybrid} ::: hardware
        MIX_I((Mixer I)) ::: rfpath
        MIX_Q((Mixer Q)) ::: rfpath
        
        PLL -- LO Signal --> SPLIT
        SPLIT -- 0° --> MIX_I
        SPLIT -- 90° --> MIX_Q
        LNA --> MIX_I
        LNA --> MIX_Q
    end

    %% 4. Baseband Filters
    subgraph BB [4. Baseband Filtering]
        HPF[High-Pass Filter<br>Blocks DC/Leakage] ::: baseband
        LPF[Low-Pass Filter<br>Noise Shaping] ::: baseband
        VGA[Variable Gain Amp<br>Scale to ADC] ::: baseband

        MIX_I -- I_raw --> HPF
        MIX_Q -- Q_raw --> HPF
        HPF --> LPF
        LPF --> VGA
    end

    %% 5. ADC & DSP
    subgraph DSP_BLOCK [5. ADC & Digital Processing]
        ADC[Dual 12-bit ADC<br>500 kSps] ::: dsp
        WIN[2D Hann Window] ::: dsp
        IFFT[Range IFFT<br>16x Zero-Padded] ::: dsp
        FFT[Doppler FFT<br>Across Sweeps] ::: dsp
        PEAK[2D Peak Detection &<br>Parabolic Interpolation] ::: dsp
        OUT([Range &<br>Velocity]) ::: dsp

        VGA -- I_vga, Q_vga --> ADC
        ADC -- Complex Matrix<br>64 x 32 --> WIN
        WIN --> IFFT
        IFFT --> FFT
        FFT --> PEAK
        PEAK --> OUT
    end
