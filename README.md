# Paper
- __Title:__ Fast Bi-layer Neural Synthesis of One-Shot Realistic Head Avatars
- __Authors/Institution:__ Egor Zakharov, Aleksei Ivakhnenko, Aliaksandra Shysheya, Victor Lempitsky -  Samsung AI Center.
- __Link:__ https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123570511.pdf
- __Code:__ https://github.com/saic-violet/bilayer-model
- __Key-words:__ Neural Avatars, Talking Heads, Neural Rendering, Head Synthesis, Head Animation, ECCV 2020
- __Year:__ 2020

# Summary
- What

  - They propose a system that can create head avatars from a single image (one-shot setting).
  - The key idea and novelty of this work is that they split a heavy generator network, which is run for each frame during test time, into two parts:
    1) *Heavy texture generator network* that is run at the initialization (once for each identity).
    2) The second one is the *lightweight inference network* that is run once per frame.
  - In such way they are able to minimize the inference time, thus making this set-up be possible to use in the real-world applications.
  - This method achieves a significant inference speedup over previous neural head avatar models.
  - Also, it supports real-time rendering for smartphone and tablet GPUs, which wasn't possible for the previous head avatar models for a given visual quality and resolution.

- How



  - Architecture
    - The whole architecture consists of 4 networks, each of which is trained in the end-to-end fashion: *the embedder network*, *the texture generator network*, *the inference generator network* and the *discriminator network*.
      1) At first, using *the embedder network*  <img src="https://latex.codecogs.com/gif.latex?E\Big(x^i(s),&space;y^i(s)\Big)"> , they encode the source frame and the corresponding landmark image into the embeddings.
          - These embeddings are then used to initialize adaptive parameters of both inference and texture generators.
      2) After initialization of adaptive parameters, *texture generator* <img src="https://latex.codecogs.com/gif.latex?G_{tex}\Big(\{e^{i}_{k}(s)&space;\}\Big)"> predicts a high-frequency texture. 
          - This texture is initialized once per identity and is aimed to be pose-invariant.
      3) After initialization of adaptive parameters, the *inference generator* <img src="https://latex.codecogs.com/gif.latex?G_{inf}\Big(y^{i}(t),&space;\{e^{i}_{k}(s)\}&space;\Big)"> takes target keypoints as an input and predicts low-frequency component <img src="https://latex.codecogs.com/gif.latex?\hat{x}_{LF}^{i}(t)">.
          - This low-frequency component encodes basic facial features, skin color and lighting.
          - After that, the high-frequency component <img src="https://latex.codecogs.com/gif.latex?\hat{x}_{HF}^{i}(t)"> is obtained by warping previously predicted static texture.
          - As a final step, low-frequency and high-frequency components are added together <img src="https://latex.codecogs.com/gif.latex?\hat{x}^{i}(t)&space;=&space;\hat{x}_{LF}^{i}(t)&space;&plus;&space;\hat{x}_{HF}^{i}(t)"> to produce the result image.
      4) Also, *the discriminator network* <img src="https://latex.codecogs.com/gif.latex?D\Big(x^{i}(t),&space;y^{i}(t)&space;\Big)"> is a conditional relativistic PatchGAN and is used to evaluate the generative performance of all models by predicting realism scores <img src="https://latex.codecogs.com/gif.latex?s^{i}(t)">
      
    - Visualization:
    <br/><br/>
    <img src="https://saic-violet.github.io/bilayer-model/assets/scheme.png">
  
    - A visualization of each individual output produced by the model:
    <br/><br/>
    ![visuals_comp](https://user-images.githubusercontent.com/22610398/101680600-99213700-3a69-11eb-9bfc-71a2becc30a7.gif)
  
  - Training process
  
    - Multiple losses are used for training:
    
      - Adversarial loss
        - This is a main loss function and it is responsible for the realism of the output.
        - It is optimized by both generators, the embedder and the discriminator networks.
      
      - Pixelwise loss
        - This loss is responsible for the preservation of the source lightning condition.
      <p align="center">
        <img src="https://latex.codecogs.com/gif.latex?L^{G}_{pix}&space;=&space;\frac{1}{HW}\|\hat{x}^{i}_{LF}(t)&space;-&space;x^{i}(t)\|_{1}">
      </p>
      
      - Perceptual loss
        - This loss is responsible for matching the source identity in the output.
        - To calculate the perceptual loss, they use the stop-gradient operator, which allows to prevent the gradient flow into a low-frequency component.
      
      - Texture mapping regularization loss
        -  This loss adds robustness to the random initialization of the model.
      <p align="center">
        <img src="https://latex.codecogs.com/gif.latex?L^{G}_{reg}&space;=&space;\frac{1}{HW}\|w^{i}(t)&space;-&space;I\|_{1}">
      </p>
      
    - Texture enhancement network is used to fine-tune the generator weights in the few-shot training set in order to minimize the identity gap.
      - The Updater network accepts the current state of the texture and the guiding gradients to produce the next state. 
      - The guiding gradients are obtained by reconstructing the source image from the current state of the texture and matching it to the ground-truth via a lightweight updater loss.
      - The gradients from this loss are then backpropagated through all M copies of the updater network.
      - This step leads to significant improvement in realism and identity preservation of the synthesized images.
      - The visualization of the process:
    <br/><br/>
    <img src="https://user-images.githubusercontent.com/22610398/101676885-76d8ea80-3a64-11eb-8713-d066b8429ea1.png">
  
  - Implementation details
    - All networks consist of pre-activation residual blocks with LeakyReLU activations.
    - A minimum number of features in these blocks is 64, and a maximum is 512.
    - Batch normalization is used in all the networks except for the embedder and the texture updater.
    - Gradient descent is done simultaneously on parameters of the generator networks and the discriminator using Adam optimizer and learning rate of 0.0002.
    - Batch normalization statistics are synchronized across all GPUs during training.
    - Spectral normalization is also applied in all linear and convolutional layers of all networks.
    - All models are trained on 8 NVIDIA P40 GPUs with the batch size of 48 for the base model, and a batch size of 32 for the updater model.

- Results
  - Metrics used for evaluation:
    - *LPIPS* (Learned perceptual image patch similarity) - measures overall predicted image similarity to ground truth - *smaller is better*.
    - *CSIM* - cosine similarity between the embedding vectors of a state-of-the-art face recognition network CSIM, calculated using the synthesized and the target images. This metric evaluates the identity mismatch - *higher is better*.
    - *NME* (Normalized mean error) - Normalized mean error of the head pose in the synthesized image - *smaller is better*.
    - *MACs* (Multiply-accumulate operations) - Measure the complexity of each method, specifically the inference stage.
    
  - For the evaluation was used the subset of 50 test videos with different identities from VoxCeleb2 dataset (contains 140697 videos of 5994 different people).
  - The comparison is done against 2 state-of-the-art systems:
    - *Few-shot Vid-to-Vid* - state-of-the-art video-to-video translation system, which has also been successfully applied to this problem.
    - *First Order Motion Model* (FOMM) is a general motion transfer system that does not use precomputed keypoints, but can also be used as an avatar.
  - Results against state-of-the-art models by the abovementioned metrics:
    <br/><br/>
    <img src="https://user-images.githubusercontent.com/22610398/101679606-4004d380-3a68-11eb-8098-d4190d80ef6a.png">
  - The graphs reveal that the proposed model outperforms previous SOTA systems and achieves state-of-the-art quality for the smaller model complexity.
  - Self-driving results:
  <br/><br/>
  ![visuals_self_smaller](https://user-images.githubusercontent.com/22610398/101680229-0ed8d300-3a69-11eb-9ad4-8e45db6527a0.gif)
