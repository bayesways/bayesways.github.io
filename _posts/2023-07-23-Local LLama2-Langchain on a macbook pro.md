---
layout: post
date: 2023-07-24
title: Local LLama2+Langchain on a macbook pro
giscus_comments: true
related_posts: false
tags: llm
---

In this post I will show how to build a simple LLM chain that runs completely locally on your macbook pro. Will use the latest  Llama2 models with Langchain. 

# Run locally on your Macbook Pro

Create a directory to put all the models and code notebooks in
```shell
mkdir llama2
cd llama2
```

then follow the instructions by  [Suyog Sonwalkar](https://blog.lastmileai.dev/run-llama-2-locally-in-7-lines-apple-silicon-mac-c3f46143f327) copied here for convenience: 
```shell
git clone https://github.com/ggerganov/llama.cpp.git  
cd llama.cpp  
curl -L https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGML/resolve/main/llama-2-7b-chat.ggmlv3.q4_K_M.bin --output ./models/llama-2-7b-chat.ggmlv3.q4_K_M.bin   
LLAMA_METAL=1 make
```

This will save the local model at the following path (we will need this path later)
```
"llama2/llama.cpp/models/llama-2-7b-chat.ggmlv3.q4_K_M.bin",
```

You can test the model works by telling it "recipe for avocado toast". To do that go to the `llama.cpp` directory (if you are not already) and do
```shell
./main -m ./models/llama-2-7b-chat.ggmlv3.q4_K_M.bin -n 1024 -ngl 1 -p "recipe for avocado toast"
```

You should get something like this: 
```shell
main: build = 893 (4f06592)
main: seed  = 1690215975
llama.cpp: loading model from ./models/llama-2-7b-chat.ggmlv3.q4_K_M.bin
llama_model_load_internal: format     = ggjt v3 (latest)
llama_model_load_internal: n_vocab    = 32000
llama_model_load_internal: n_ctx      = 512
llama_model_load_internal: n_embd     = 4096
llama_model_load_internal: n_mult     = 256
llama_model_load_internal: n_head     = 32
llama_model_load_internal: n_head_kv  = 32
llama_model_load_internal: n_layer    = 32
llama_model_load_internal: n_rot      = 128
llama_model_load_internal: n_gqa      = 1
llama_model_load_internal: n_ff       = 11008
llama_model_load_internal: freq_base  = 10000.0
llama_model_load_internal: freq_scale = 1
llama_model_load_internal: ftype      = 15 (mostly Q4_K - Medium)
llama_model_load_internal: model size = 7B
llama_model_load_internal: ggml ctx size =    0.08 MB
llama_model_load_internal: mem required  = 4193.33 MB (+  256.00 MB per state)
llama_new_context_with_model: kv self size  =  256.00 MB
ggml_metal_init: allocating
ggml_metal_init: using MPS
ggml_metal_init: loading '/Users/Konstantinosvamvourellies/llama2/llama.cpp/ggml-metal.metal'
ggml_metal_init: loaded kernel_add                            0x107007530
ggml_metal_init: loaded kernel_add_row                        0x107007d60
ggml_metal_init: loaded kernel_mul                            0x107008280
ggml_metal_init: loaded kernel_mul_row                        0x1070088b0
ggml_metal_init: loaded kernel_scale                          0x107008dd0
ggml_metal_init: loaded kernel_silu                           0x1070092f0
ggml_metal_init: loaded kernel_relu                           0x107009810
ggml_metal_init: loaded kernel_gelu                           0x107009d30
ggml_metal_init: loaded kernel_soft_max                       0x10700a3e0
ggml_metal_init: loaded kernel_diag_mask_inf                  0x10700aa40
ggml_metal_init: loaded kernel_get_rows_f16                   0x10700b0c0
ggml_metal_init: loaded kernel_get_rows_q4_0                  0x10700b8b0
ggml_metal_init: loaded kernel_get_rows_q4_1                  0x10700bf30
ggml_metal_init: loaded kernel_get_rows_q2_K                  0x10700c5b0
ggml_metal_init: loaded kernel_get_rows_q3_K                  0x10700cc30
ggml_metal_init: loaded kernel_get_rows_q4_K                  0x10700d2b0
ggml_metal_init: loaded kernel_get_rows_q5_K                  0x10700d930
ggml_metal_init: loaded kernel_get_rows_q6_K                  0x10700dfb0
ggml_metal_init: loaded kernel_rms_norm                       0x10700e670
ggml_metal_init: loaded kernel_norm                           0x10700ee90
ggml_metal_init: loaded kernel_mul_mat_f16_f32                0x10700f6f0
ggml_metal_init: loaded kernel_mul_mat_q4_0_f32               0x10700fdb0
ggml_metal_init: loaded kernel_mul_mat_q4_1_f32               0x107010470
ggml_metal_init: loaded kernel_mul_mat_q2_K_f32               0x107010cb0
ggml_metal_init: loaded kernel_mul_mat_q3_K_f32               0x107011370
ggml_metal_init: loaded kernel_mul_mat_q4_K_f32               0x107011a30
ggml_metal_init: loaded kernel_mul_mat_q5_K_f32               0x1070120d0
ggml_metal_init: loaded kernel_mul_mat_q6_K_f32               0x107012bd0
ggml_metal_init: loaded kernel_rope                           0x1070130f0
ggml_metal_init: loaded kernel_alibi_f32                      0x1070139b0
ggml_metal_init: loaded kernel_cpy_f32_f16                    0x107014240
ggml_metal_init: loaded kernel_cpy_f32_f32                    0x107014ad0
ggml_metal_init: loaded kernel_cpy_f16_f16                    0x107015240
ggml_metal_init: recommendedMaxWorkingSetSize = 10922.67 MB
ggml_metal_init: hasUnifiedMemory             = true
ggml_metal_init: maxTransferRate              = built-in GPU
llama_new_context_with_model: max tensor size =   102.54 MB
ggml_metal_add_buffer: allocated 'data            ' buffer, size =  3891.69 MB, ( 3892.14 / 10922.67)
ggml_metal_add_buffer: allocated 'eval            ' buffer, size =    10.00 MB, ( 3902.14 / 10922.67)
ggml_metal_add_buffer: allocated 'kv              ' buffer, size =   258.00 MB, ( 4160.14 / 10922.67)
ggml_metal_add_buffer: allocated 'scr0            ' buffer, size =   132.00 MB, ( 4292.14 / 10922.67)
ggml_metal_add_buffer: allocated 'scr1            ' buffer, size =   160.00 MB, ( 4452.14 / 10922.67)

system_info: n_threads = 6 / 10 | AVX = 0 | AVX2 = 0 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | FMA = 0 | NEON = 1 | ARM_FMA = 1 | F16C = 0 | FP16_VA = 1 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 0 | VSX = 0 |
sampling: repeat_last_n = 64, repeat_penalty = 1.100000, presence_penalty = 0.000000, frequency_penalty = 0.000000, top_k = 40, tfs_z = 1.000000, top_p = 0.950000, typical_p = 1.000000, temp = 0.800000, mirostat = 0, mirostat_lr = 0.100000, mirostat_ent = 5.000000
generate: n_ctx = 512, n_batch = 512, n_predict = 1024, n_keep = 0


 recipe for avocado toast with fresh herbs and lemon

This is a simple and delicious recipe for avocado toast that incorporates fresh herbs and a squeeze of lemon. The addition of the herbs gives the dish a bright, refreshing flavor that complements the richness of the avocado.
Ingredients:

* 2 slices of bread (preferably whole wheat or whole grain)
* 1 ripe avocado, mashed
* Fresh herbs (such as parsley, basil, or cilantro) chopped
* Salt and pepper to taste
* Lemon wedges (optional)
Instructions:

1. Toast the bread until it is lightly browned.
2. Spread the mashed avocado on top of the toasted bread.
3. Sprinkle the chopped fresh herbs over the avocado.
4. Season with salt and pepper to taste.
5. Squeeze a slice of lemon over the avocado toast, if desired.
6. Serve immediately and enjoy!

This recipe is easy to make and can be customized to your liking by using different types of bread or herbs. The lemon adds a nice tanginess to the dish, but feel free to omit it if you prefer. Enjoy! [end of text]

llama_print_timings:        load time =  5282.86 ms
llama_print_timings:      sample time =   200.57 ms /   308 runs   (    0.65 ms per token,  1535.59 tokens per second)
llama_print_timings: prompt eval time =   637.46 ms /     9 tokens (   70.83 ms per token,    14.12 tokens per second)
llama_print_timings:        eval time =  9804.24 ms /   307 runs   (   31.94 ms per token,    31.31 tokens per second)
llama_print_timings:       total time = 10668.64 ms
ggml_metal_free: deallocating
```


The model output is this: 
```
 recipe for avocado toast with fresh herbs and lemon

This is a simple and delicious recipe for avocado toast that incorporates fresh herbs and a squeeze of lemon. The addition of the herbs gives the dish a bright, refreshing flavor that complements the richness of the avocado.
Ingredients:

* 2 slices of bread (preferably whole wheat or whole grain)
* 1 ripe avocado, mashed
* Fresh herbs (such as parsley, basil, or cilantro) chopped
* Salt and pepper to taste
* Lemon wedges (optional)
Instructions:

1. Toast the bread until it is lightly browned.
2. Spread the mashed avocado on top of the toasted bread.
3. Sprinkle the chopped fresh herbs over the avocado.
4. Season with salt and pepper to taste.
5. Squeeze a slice of lemon over the avocado toast, if desired.
6. Serve immediately and enjoy!

This recipe is easy to make and can be customized to your liking by using different types of bread or herbs. The lemon adds a nice tanginess to the dish, but feel free to omit it if you prefer. Enjoy! [end of text]
```


In this example we installed the LLama2-7B param model for chat. Later I will show how to do the same for the bigger Llama2 models. To see all the LLM model versions that Meta has released on hugging face go [here](https://huggingface.co/meta-llama). Note that to use any of these models from hugging face you'll need to request approval using this [form](https://ai.meta.com/resources/models-and-libraries/llama-downloads/). You can do that following this [demo](https://twitter.com/jamescalam/status/1682766618831777794?s=61&t=Pw-aY--IwGlNpRBvUW-P9g) by James Briggs.

To run the model locally though you'll need the quantized versions of the model (in order to fit within the limitations of your macbook pro). These versions have been made available by [TheBloke](https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGML/tree/main) (based on the work by [ggerganov](https://github.com/ggerganov/llama.cpp)), and that's what we are using here. 


# Run with Langchain

To use your local model with Langchain  follow the langchain documentation [here](https://python.langchain.com/docs/use_cases/question_answering/local_retrieval_qa#llama-v2). 

Note that you'll need to pass the correct path to your model bin file in the LLamaCpp `model_path` param . For me this path was `"../llama.cpp/models/llama-2-7b-chat.ggmlv3.q4_K_M.bin"` as shown below:
```python
# Make sure the model path is correct for your system!
llm = LlamaCpp(
    model_path="../llama.cpp/models/llama-2-7b-chat.ggmlv3.q4_K_M.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    n_ctx=2048,
    f16_kv=True,  # MUST set to True, otherwise you will run into problem after a couple of calls
    callback_manager=callback_manager,
    verbose=True,
)
```


# Performance 
I was able to run it on my machine - see this [repo](https://github.com/bayesways/local_llama2_demo) for my setup. 
I run it on a Macbook pro with an M2 Pro chip 16G RAM. The model run considerably slower with Langchain versus when prompted from the terminal. To understand why I compared the parameters used in the calls made by Langchain's `LlamaCpp` and the terminal call `main` . The only difference was the `llama_model_load_internal: mem required`. 

In the terminal I got 
```
llama_model_load_internal: mem required  = 4193.33 MB (+  256.00 MB per state)
```

In Langchain I got
```
llama_model_load_internal: mem required  = 5563.32 MB (+ 1026.00 MB per state)
```

Will be looking into this and write an update. 

# Running other LLama model versions

LLama has released different versions of the models, as discussed. However to run them locally you'll need the quantized versions provided by independent developers such as TheBloke. 

To download other versions you'll need to
 - find the quantized version you'll need, for example go [here](https://huggingface.co/TheBloke/Llama-2-70B-GGML) for the 70B param GGML model version
 - adjust the names of the models in the commands used, for example replace `llama-2-7b-chat.ggmlv3.q4_K_M.bin`  with `llama-2-70b.ggmlv3.q4_0.bin` 

# Run in Google Colab

If you don't have a macbook with a M2 chip or want faster performance you can run llama2 with langchain in a google colab [notebook](https://colab.research.google.com/drive/11o2k1iyDCQvhp2z_0NrTlAfgKy01P60Z?usp=sharing). I've taken the instructions from this [demo](https://twitter.com/jamescalam/status/1682766618831777794?s=61&t=Pw-aY--IwGlNpRBvUW-P9g) by James Briggs with minimal adaptations. The notebook runs from start to finish in ~ 10 mins.

Here we are running the 13b parameter version to avoid hitting the GPU memory limits of the free version. 

You'll need 
 - To apply to use LLama2 and get approved (go [here](https://ai.meta.com/resources/models-and-libraries/llama-downloads/) and register with an email same as the one in your hugging face account)
 - Hugging face api key
 - Hugging face to match your Meta approval with your Hugging face email 
 
Save the Hugging face api key in a file `secret_api_keys.env` which you'll save in your top directory of the google drive. 

## References

- Llama2+Langchain [demo](https://twitter.com/jamescalam/status/1682766618831777794?s=61&t=Pw-aY--IwGlNpRBvUW-P9g) by James Briggs 
- Run LLama2 on macbook pro based on three similar instructions by [Adrien Brault](https://twitter.com/AdrienBrault/status/1681503574814081025),  [Suyog Sonwalkar](https://blog.lastmileai.dev/run-llama-2-locally-in-7-lines-apple-silicon-mac-c3f46143f327) or [Abhishek Thakur](https://www.youtube.com/watch?v=Kg588OVYTiw)
- Langchain [documentation](https://python.langchain.com/docs/use_cases/question_answering/local_retrieval_qa) for locally run models
