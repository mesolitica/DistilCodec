<div align="center">
    <h1>
    DistilCodec
    </h1>
    <p>
    <b><em>DistilCodec: A Single Codebook Audio Codec For Universal Audio</em></b>
   </p>
    <p>
    </p>
    </p>
    <a href="https://arxiv.org/abs/2505.17426" style="color:red">Paper </a> |  
    <a href="https://huggingface.co/IDEA-Emdoor/DistilCodec-v1.0" style="color:#FFD700">HuggingFace Model</a> |
    <a href="https://github.com/IDEA-Emdoor-Lab/DistilCodec" style="color:gray">Code</a>
     <p>
        <img src="./data/figures/idea_logo.png" alt="Institution 1" style="width: 200px; height: 60px;">
     </p>
     <p>
        <img src="./data/figures/yidao_logo.png" alt="Institution 2" style="width: 200px; height: 60px;">
        <img src="./data/figures/yijiayiban.png" alt="Institution 3" style="width: 200px; height: 60px;">
    </p>
</div>


# 🔥 News
- *2025.05.26*: We release DistilCodec-v1.0 checkpoint on [huggingface](https://huggingface.co/IDEA-Emdoor/DistilCodec-v1.0).
- *2025.05.26*: The paper is available on [arxiv](https://arxiv.org/abs/2505.17426).
- *2025.05.23*: We submit paper to arxiv.

## Introduction of DistilCodec
The Joint Laboratory of International Digital Economy Academy (IDEA) and Emdoor, in collaboration with Emdoor Information Technology Co., Ltd., and Shenzhen Yijiayiban Information Technology Co., Ltd, has launched DistilCodec - A Single-Codebook Neural Audio Codec (NAC) with 32768 codes trained on universal audio. We also trained a TTS based on DistilCodec which called [UniTTS](https://github.com/IDEA-Emdoor-Lab/UniTTS). To better leverage the universal audio reconstruction capability of DistilCodec, UniTTS incorporates the <em>universal audio autoregressive task</em> in ALM-Pretrain. For details, please refer to our [paper](https://arxiv.org/abs/2505.17426). The foundational network architecture of DistilCodec adopts an Encoder-VQ-Decoder framework similar to that proposed in Soundstream. The encoder employs a ConvNeXt-V2 structure, while the vector quantization module implements the GRFVQ scheme. The decoder employs a ConvTranspose1d based architectural configuration similar to HiFiGAN The training methodology of DistilCodec follows a similar approach to HiFiGAN, incorporating three types of discriminators: Multi-Period Discriminator (MPD), Multi-Scale Discriminator (MSD), and Multi-STFT Discriminator (MSFTFD). Here is the architecture of Distilcodec:
![The Architecture of DistilCodec](./data/figures/distilcodec_architecture.jpg)
Distribution of DistilCodec training data is shown in below table:
| **Data Category**           | **Data Size (in hours)** |
|-----------------------------|--------------------------|
| Chinese Audiobook           | 38000                    |
| Chinese Common Audio        | 20000                    |
| English Audiobook           | 10000                    |
| English Speech              | 30000                    |
| Music                       | 2000                     |
| **Total**                   | **100000**               |

### Training Schema
We have developed a novel distillation approach termed DMS (**D**istilling **M**ulti-Codebook NAC to **S**ingle-Codebook NAC) by enabling the Student NAC to inherit encoder and decoder parameters from the Teacher NAC. Based on DMS, we trained DistilCodec using universal audio datasets as training data, achieving a single codebook with a codebook size of 32,768 while maintaining codebook utilization approaching 100\%. Simultaneously, the DMS algorithm enables the dimension of the distilled Student NAC Codebook to be scaled beyond 2048. Leveraging this capability, we configured the codebook dimension to 3584, aligning with the word embedding dimension of QWen2.5-7B (3584), so we subsequently leveraged DistilCodec's codebook to initialize the audio embedding layer in [UniTTS](https://github.com/IDEA-Emdoor-Lab/UniTTS). Here is the psuedo code of DMS:
#### Algorithm DMS: Distilling Multi-Codebook NAC to Single-Codebook NAC via parameter inheritance
1. **Step 1:** Initializing *Teacher codec*:

   <img src="./data/figures/step1_formula.png" alt="Step1 formula" style="width: 50%; height: auto;" />
2. **Step 2:** *Teacher codec* training with LSGAN
3. **Step 3:** Initializing *Student codec*:

   <img src="./data/figures/step3_formula.png" alt="Step3 formula" style="width: 50%; height: auto;" />
4. **Step 4:** *Student codec* training with DLF
5. **Output:** DistilCodec = Student_codec

The parameter settings for the codebooks of Teacher Codec and Student Codec are as follows, where N-Residual indicates the number of residual layers, N-Group denotes the number of groups, N-Codes/Codebook represents the number of codes per codebook, and Dimension specifies the dimension of the codebook.
| Codec       | N-Residual | N-Group | N-Codes/Codebook | Dimension |
|-------------|------------|---------|------------------|-----------|
| Teacher-Codec | 8          | 4       | 1024             | 512       |
| Student-Codec | 1          | 1       | 32768            | 3584      |

### Evaluation and Demos
The second row of the table demonstrates the codebook utilization and perplexity (PPL) of DistilCodec evaluated on LibriSpeech-Test-Clean. Given DistilCodec's capability to process universal audio, we have constructed an integrated test set comprising speech, audiobook, and music samples for evaluating codebook utilization and PPL in universal audio scenarios. As shown in the table, DistilCodec achieves near-optimal codebook utilization (approaching 100%) across both datasets, accompanied by notably high PPL values (the theoretical maximum PPL equals the codebook size, which is 32,768). These results substantiate DistilCodec's superior audio reconstruction capabilities in universal audio applications.
| Dataset              | Codebook Usage(%)↑ | Codebook PPL↑ |
|-----------------------|---------------------|---------------|
| LibriSpeech-Clean-Test| 98.2                | 21660.5       |
| Universal-Audio-Test  | 99.9                | 26999.0       |

Additionally, we conducted a comprehensive comparative analysis of DistilCodec’s speech reconstruction capabilities using the LibriSpeech-Clean-Test benchmark. 
| Model             | Codebook Size | Nq | Token Rate (TPS) | Bandwidth (bps) | STOI ↑ | PESQ ↑ | UTMOS ↑ |
|-------------------|---------------|----|------------------|----------------|--------|--------|--------|
| Encodec           | 1024          | 8  | 600              | 6000           | 0.94   | 2.75   | 3.07   |
| DAC               | 1024          | 12 | 600              | 6000           | 0.95   | 4.01   | 4.00   |
| Encodec           | 1024          | 2  | 150              | 1500           | 0.84   | 1.56   | 1.58   |
| Mimi              | 2048          | 8  | 100              | 1100           | 0.91   | 2.25   | 3.56   |
| BigCodec          | 8192          | 1  | 80               | 1040           | 0.94   | 2.68   | 4.11   |
| DAC               | 1024          | 2  | 100              | 1000           | 0.73   | 1.14   | 1.29   |
| SpeechTokenizer   | 1024          | 2  | 100              | 1000           | 0.77   | 1.25   | 2.28   |
| X-codec           | 1024          | 2  | 100              | 1000           | 0.86   | 2.33   | 4.21   |
| WavTokenizer      | 4096          | 1  | 75               | 900            | 0.89   | 2.14   | 3.94   |
| X-codec2          | 65536         | 1  | 50               | 800            | 0.92   | 2.43   | 4.13   |
| StableCodec       | 15625         | 2  | 50               | 697            | 0.91   | 2.24   | 4.23   |
| Single-Codec      | 8192          | 1  | 23.4             | 304            | 0.86   | 1.88   | 3.72   |
| BiCodec           | 8192          | 1  | 50               | 650            | 0.92   | 2.51   | 4.18   |
| DistilCodec       | 32768         | 1  | 93               | 1300           | 0.93   | 2.02   | 3.75   |

Since DistilCodec was trained on universal audio, we first employed UTMOS for automatic quality assessment. However, the universal audio test set received an unreliable low score (1.89), indicating UTMOS's inadequacy for universal audio evaluation. We therefore conducted a Mean Opinion Score (MOS) evaluation, the results are shown:
| Assessment Items          | Reconstructed | Original    |
|---------------------------|------------------|-------|
| Speech Clarity            | 4.689            | 4.945 |
| Background Audio Clarity  | 4.768            | 4.927 |
| Average Score             | 4.728            | 4.936|

The MOS evaluation dataset comprises original audio samples stored in the [Original Audios](./data/org_audios/) directory and corresponding reconstructed samples generated by DistilCodec in the [Reconstructed Audios](./data/gen_audios). Below are comparative analyses between selected original and reconstructed audio pairs, due to some compatibility issues, the audio preview in the table below may have display problems. You can clone the project locally and view it using VS Code's Markdown preview.
| Category        | Original Audio | Reconstructed Aduio   |
|---------------------------|------------------|-------|
| Chinese Audio    |<audio controls src="./data/org_audios/0b0c96e3-e2ae-45a3-9488-806cd719517b_0175.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/0b0c96e3-e2ae-45a3-9488-806cd719517b_0175.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/0b0c96e3-e2ae-45a3-9488-806cd719517b_0175.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/0b0c96e3-e2ae-45a3-9488-806cd719517b_0175.wav">audio file</a>。</audio>|
| Chinese Audio    |<audio controls src="./data/org_audios/0d28f03f-70c8-4180-ba1c-37b167aa9447_0074.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/0d28f03f-70c8-4180-ba1c-37b167aa9447_0074.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/0d28f03f-70c8-4180-ba1c-37b167aa9447_0074.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/0d28f03f-70c8-4180-ba1c-37b167aa9447_0074.wav">audio file</a>。</audio>|
| Chinese Audio    |<audio controls src="./data/org_audios/0eff38a1-3c9c-4a33-9be9-896614417d3f_0081.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/0eff38a1-3c9c-4a33-9be9-896614417d3f_0081.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/0eff38a1-3c9c-4a33-9be9-896614417d3f_0081.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/0eff38a1-3c9c-4a33-9be9-896614417d3f_0081.wav">audio file</a>。</audio>|
| English Audio    |<audio controls src="./data/org_audios/f0b1da30-ad19-4619-8aee-4b5c6d8c4acf_POD0000003287_S0000341.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/f0b1da30-ad19-4619-8aee-4b5c6d8c4acf_POD0000003287_S0000341.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/f0b1da30-ad19-4619-8aee-4b5c6d8c4acf_POD0000003287_S0000341.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/f0b1da30-ad19-4619-8aee-4b5c6d8c4acf_POD0000003287_S0000341.wav">audio file</a>。</audio>|
| English Audio    |<audio controls src="./data/org_audios/0016.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/0016.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/0016.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/0016.wav">audio file</a>。</audio>|
| English Audio    |<audio controls src="./data/org_audios/2f7f51c9-c514-4a23-8c31-d032c929df46_YOU0000006574_S0000379.wav">Your browser does not support audio playback. Please download <a href="./data/org_audios/2f7f51c9-c514-4a23-8c31-d032c929df46_YOU0000006574_S0000379.wav">audio file</a>。</audio>|<audio controls src="./data/gen_audios/2f7f51c9-c514-4a23-8c31-d032c929df46_YOU0000006574_S0000379.wav">Your browser does not support audio playback. Please download <a href="./data/gen_audios/2f7f51c9-c514-4a23-8c31-d032c929df46_YOU0000006574_S0000379.wav">audio file</a>。</audio>|

For additional comparative audio examples, please use our MOS evaluation tool:
```bash
python codec_evaluation_gradio.py
```
Upon launching the system, the interface displays the following components: Model1 represents the original audio, while Model2 corresponds to the audio reconstructed by DistilCodec.
![DistilCodec MOS Tool](./data/figures/distilcodec_mos.png)

If you want to perform a benchmark evaluation on LibriSpeech-test, you can follow these steps:
- *Eval Config*: Modify the values of parameters in [Eval Cofig](./scripts/examples/evaluation/libri_test_clean.json), such as filelist_path, save_dir.
- *Eval Shell*: Modify the values of parameters in [Eval Shell](./scripts/examples/evaluation/libri_test_clean_eval.sh).
- *Execute Shell*: Run the eval shell.

## Installation of DistilCodec

```bash
pip3 install git+https://github.com/mesolitica/DistilCodec
```

## Inference of DistilCodec

### Part1:  Generate audio tokens using DistilCodec

```python
# wget https://huggingface.co/IDEA-Emdoor/DistilCodec-v1.0/resolve/main/g_00204000

from distilcodec import DistilCodec, demo_for_generate_audio_codes

codec_model_config_path='configs/model_config.json'
codec_ckpt_path = 'g_00204000'

codec = DistilCodec.from_pretrained(
    config_path=codec_model_config_path,
    model_path=codec_ckpt_path,
    use_generator=True,
    is_debug=False).eval()

audio_path = 'test.mp3'
audio_tokens = demo_for_generate_audio_codes(
    codec, 
    audio_path, 
    target_sr=24000, 
    plus_llm_offset=False
)
print(audio_tokens)
```

### Part2: Reconstruct audio with audio tokens generated from DistilCodec

```python
y_gen = codec.decode_from_codes(
    audio_tokens, 
    minus_token_offset=False
)
```

## Available DistilCodec models

|Model Version| Huggingface |  Corpus  |  Token/s  | Domain |
|-----------------------|---------|---------------|---------------|-----------------------------------|
| DistilCodec-v1.0 | [HuggingFace](https://huggingface.co/IDEA-Emdoor/DistilCodec-v1.0) | Universal Audio | 93 | Universal Audio |

## References
The overall training pipeline of DistilCodec draws inspiration from AcademiCodec, while its encoder and decoder design is adapted from fish-speech. The Vector Quantization (VQ) component implements GRFVQ using the vector-quantize-pytorch framework. These three exceptional works have provided invaluable assistance in our implementation of DistilCodec. Below are links to these reference projects:

[1][vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch)

[2][AcademiCodec](https://github.com/moewiee/hificodec)

[3][fish-speech](https://github.com/fishaudio/fish-speech)


## Citation

If you find our work useful in your research, please cite our work:

```
@misc{wang2025unittsendtoendttsdecoupling,
      title={UniTTS: An end-to-end TTS system without decoupling of acoustic and semantic information}, 
      author={Rui Wang and Qianguo Sun and Tianrong Chen and Zhiyun Zeng and Junlong Wu and Jiaxing Zhang},
      year={2025},
      eprint={2505.17426},
      archivePrefix={arXiv},
      primaryClass={cs.SD},
      url={https://arxiv.org/abs/2505.17426}, 
}
```


## Disclaimer

DistilCodec provides the capability of universal audio discretion only for academic research purposes. We encourage the community to uphold safety and ethical principles in AI research and applications.

Important Notes:

- Compliance with the model's open-source license is mandatory.

- Unauthorized voice replication applications are strictly prohibited.

- Developers bear no responsibility for any misuse of this model.


## License
<a href="https://arxiv.org/abs/2505.17426">UniTTS: An end-to-end TTS system without decoupling of acoustic and semantic information</a> © 2025 by <a href="https://creativecommons.org">Rui Wang, Qianguo Sun, Tianrong Chen, Zhiyun Zeng, Junlong Wu, Jiaxing Zhang</a> is licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/">CC BY-NC-ND 4.0</a><img src="https://mirrors.creativecommons.org/presskit/icons/cc.svg" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/by.svg" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/nc.svg" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/nd.svg" style="max-width: 1em;max-height:1em;margin-left: .2em;">