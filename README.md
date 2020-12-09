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
    1) Heavy network that is run at the initialization (once for each identity).
    2) The second one is the lightweight inference network that is run once per frame.
  - In such way they are able to minimize the inference time, thus making this set-up be possible to use in the real-world applications.
  - This method achieves a significant inference speedup over previous neural head avatar models.
  - Also it supports real-time rendering for smartphone and tablet GPUs, which wasn't possible for the previous head avatar models for a given visual quality.
  
  

- How
- Results
