# LayerSkip
[![arXiv](https://img.shields.io/badge/arXiv-2404.16710-b31b1b.svg)](https://arxiv.org/abs/2404.16710) <a href='https://huggingface.co/collections/facebook/layerskip-666b25c50c8ae90e1965727a'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-blue'></a> [![License: CC BY-NC](https://img.shields.io/badge/License-CC_BY--NC-lightgrey.svg)](./LICENSE) [![YouTube](https://badges.aleen42.com/src/youtube.svg)](https://www.youtube.com/watch?v=oPxdfVVmLP8)


This code base is the implementation of [LayerSkip: Enabling Early Exit Inference and Self-Speculative Decoding](https://arxiv.org/abs/2404.16710).


## Getting Started
- Clone repo:
```console
$ git clone git@github.com:facebookresearch/LayerSkip.git
$ cd LayerSkip
```

- Setup environment:
```console
$ conda create --name layer_skip python=3.10
$ conda activate layer_skip

$ pip install -r requirements.txt
```

## Run
AR:
```console
$ torchrun generate.py --model facebook/layerskip-llama2-7B \
    --sample True \
    --max_steps 512

$ torchrun generate.py --model facebook/layerskip-llama2-13B \
    --sample True \
    --max_steps 512
```

SS:
```console
$ torchrun generate.py --model facebook/layerskip-llama2-7B \
    --sample True \
    --max_steps 512 \
    --generation_strategy self_speculative \
    --num_speculations 6 \
    --exit_layer 8

$ torchrun generate.py --model facebook/layerskip-llama2-13B \
    --sample True \
    --max_steps 512 \
    --generation_strategy self_speculative \
    --num_speculations 4 \
    --exit_layer 6
```

## Evaluate

- Llama 7B continual:
    - CNN/DM Summarization

        AR:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --num_samples 100 \
            --output_dir ./logs
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.1283082515001297, 'rouge-1': 0.18510770797729492, 'rouge-2': 0.08633019775152206, 'rouge-3': 0.05044793710112572, 'bleu_score': 0.0, 'exact_match': 1740.0699462890625}, 'acceptance_rate': {'mean': -1.0}, 'total_time': {'mean': 12.617329008579254}, 'time_per_token': {'mean': 0.025650737695395946}, 'tokens_per_second': {'mean': 39.04662847518921}}
        ```


        SS:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --num_samples 100 \
            --generation_strategy self_speculative \
            --num_speculations 6 \
            --exit_layer 8 \
            --output_dir ./logs
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.1364927887916565, 'rouge-1': 0.19389228522777557, 'rouge-2': 0.09072532504796982, 'rouge-3': 0.054363008588552475, 'bleu_score': 0.0, 'exact_match': 1680.0799560546875}, 'acceptance_rate': {'mean': 0.7945407949388027}, 'total_time': {'mean': 6.033521366119385}, 'time_per_token': {'mean': 0.013116696625947952}, 'tokens_per_second': {'mean': 80.12186950683594}}
        ```

    - CNN/DM Language Modeling

        AR:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_lm \
            --num_samples 10 \
            --output_dir ./logs \
            --sample True
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.13606208562850952, 'rouge-1': 0.28882747888565063, 'rouge-2': 0.058838460594415665, 'rouge-3': 0.013353409245610237, 'bleu_score': 0.0, 'exact_match': 2353.5556640625}, 'acceptance_rate': {'mean': -1.0}, 'total_time': {'mean': 11.730884949366251}, 'time_per_token': {'mean': 0.0258632300214635}, 'tokens_per_second': {'mean': 38.695508321126304}}
        ```

        SS:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_lm \
            --num_samples 10 \
            --generation_strategy self_speculative \
            --num_speculations 6 \
            --exit_layer 8 \
            --output_dir ./logs \
            --sample True
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.12094669044017792, 'rouge-1': 0.24943789839744568, 'rouge-2': 0.05234846472740173, 'rouge-3': 0.014098376035690308, 'bleu_score': 0.0, 'exact_match': 2470.89990234375}, 'acceptance_rate': {'mean': 0.4316044136881828}, 'total_time': {'mean': 7.655922102928161}, 'time_per_token': {'mean': 0.021372959669679403}, 'tokens_per_second': {'mean': 50.7227668762207}}
        ```
        Result `--temperature 0.6` (instead of 0.7):
        ```
        {'predicted_text': {'rouge-l': 0.145431250333786, 'rouge-1': 0.29284167289733887, 'rouge-2': 0.0636659488081932, 'rouge-3': 0.017067933455109596, 'bleu_score': 0.0, 'exact_match': 2270.800048828125}, 'acceptance_rate': {'mean': 0.4250859707593918}, 'total_time': {'mean': 9.79649715423584}, 'time_per_token': {'mean': 0.0218407379463315}, 'tokens_per_second': {'mean': 49.08753547668457}}
        ```
        Result `--temperature 0.6 --top_p 0.9`:
        ```
        {'predicted_text': {'rouge-l': 0.13443264365196228, 'rouge-1': 0.27407515048980713, 'rouge-2': 0.05543376877903938, 'rouge-3': 0.013210969977080822, 'bleu_score': 0.0, 'exact_match': 2435.300048828125}, 'acceptance_rate': {'mean': 0.45695279240608216}, 'total_time': {'mean': 9.832695627212525}, 'time_per_token': {'mean': 0.020543566439300776}, 'tokens_per_second': {'mean': 51.509527587890624}}
        ```
        Result `--temperature 0.6 --top_p 0.9 --top_k 0`:
        ```
        {'predicted_text': {'rouge-l': 0.13951972126960754, 'rouge-1': 0.2510159909725189, 'rouge-2': 0.05056632682681084, 'rouge-3': 0.01751687191426754, 'bleu_score': 0.0, 'exact_match': 2251.199951171875}, 'acceptance_rate': {'mean': 0.5276747912168502}, 'total_time': {'mean': 7.721955919265747}, 'time_per_token': {'mean': 0.017618346121162178}, 'tokens_per_second': {'mean': 58.09655418395996}}
        ```
        Result `--temperature 0.7 --top_p 0.9 --top_k 0`:
        ```
        {'predicted_text': {'rouge-l': 0.1246955543756485, 'rouge-1': 0.2585299015045166, 'rouge-2': 0.049484096467494965, 'rouge-3': 0.012913130223751068, 'bleu_score': 0.0, 'exact_match': 2424.0}, 'acceptance_rate': {'mean': 0.3456948846578598}, 'total_time': {'mean': 10.56712555885315}, 'time_per_token': {'mean': 0.02428342290222645}, 'tokens_per_second': {'mean': 43.264376640319824}}
        ```
        Result `--temperature 0.7 --top_p 0.95 --top_k 0`:
        ```
        {'predicted_text': {'rouge-l': 0.13389281928539276, 'rouge-1': 0.27726322412490845, 'rouge-2': 0.05858474224805832, 'rouge-3': 0.016710694879293442, 'bleu_score': 0.0, 'exact_match': 2233.0}, 'acceptance_rate': {'mean': 0.35663639903068545}, 'total_time': {'mean': 9.655048847198486}, 'time_per_token': {'mean': 0.023945740424096584}, 'tokens_per_second': {'mean': 44.228966522216794}}
        ```


        Result without sampling:
        ```
        {'predicted_text': {'rouge-l': 0.14436019957065582, 'rouge-1': 0.2590729892253876, 'rouge-2': 0.05273960903286934, 'rouge-3': 0.01946089044213295, 'bleu_score': 0.0, 'exact_match': 2361.5}, 'acceptance_rate': {'mean': 0.6267683625221252}, 'total_time': {'mean': 7.255008578300476}, 'time_per_token': {'mean': 0.01519692810252309}, 'tokens_per_second': {'mean': 70.07715606689453}}
        ```
    
    - HumanEval

        AR:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset human_eval \
            --output_dir ./logs \
            --sample True
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.1628463715314865, 'rouge-1': 0.19154858589172363, 'rouge-2': 0.0794435441493988, 'rouge-3': 0.044659942388534546, 'bleu_score': 0.0, 'exact_match': 739.768310546875}, 'acceptance_rate': {'mean': -1.0}, 'total_time': {'mean': 7.987530955454198}, 'time_per_token': {'mean': 0.025473948045656447}, 'tokens_per_second': {'mean': 39.27038741693264}}
        ```

        SS:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset human_eval \
            --generation_strategy self_speculative \
            --num_speculations 6 \
            --exit_layer 10 \
            --output_dir ./logs \
            --sample True
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.15249398350715637, 'rouge-1': 0.17897377908229828, 'rouge-2': 0.06284470856189728, 'rouge-3': 0.033304233103990555, 'bleu_score': 0.0, 'exact_match': 771.9573364257812}, 'acceptance_rate': {'mean': 0.5554095253166629}, 'total_time': {'mean': 5.63480834990013}, 'time_per_token': {'mean': 0.019379779554494634}, 'tokens_per_second': {'mean': 54.485635594623844}}
        ```
        Result `--temperature 0.6 --top_p 0.9 --top_k 0`:
        ```

        ```

    - CNN/DM Summarization (One Shot)

        AR:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --n_shot 1 \
            --num_samples 100 \
            --output_dir ./logs
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.17083728313446045, 'rouge-1': 0.23529480397701263, 'rouge-2': 0.07809153199195862, 'rouge-3': 0.03981202840805054, 'bleu_score': 0.0, 'exact_match': 236.52000427246094}, 'acceptance_rate': {'mean': -1.0}, 'total_time': {'mean': 1.0357493662834167}, 'time_per_token': {'mean': 0.03236716769635677}, 'tokens_per_second': {'mean': 31.2203493309021}}
        ```

        SS:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --n_shot 1 \
            --num_samples 100 \
            --generation_strategy self_speculative \
            --num_speculations 12 \
            --exit_layer 8 \
            --output_dir ./logs
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.17013584077358246, 'rouge-1': 0.23432669043540955, 'rouge-2': 0.07809153199195862, 'rouge-3': 0.03981202840805054, 'bleu_score': 0.0, 'exact_match': 236.6999969482422}, 'acceptance_rate': {'mean': 0.43046684432774784}, 'total_time': {'mean': 0.9261050200462342}, 'time_per_token': {'mean': 0.028940781876444818}, 'tokens_per_second': {'mean': 41.24269124031067}}
        ```

    - XSUM Summarization (Three Shot)

        AR:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset xsum_summarization \
            --n_shot 3 \
            --num_samples 100 \
            --output_dir ./logs
        ```
        Result:
        ```
        {'predicted_text': {'rouge-l': 0.22860220074653625, 'rouge-1': 0.28382208943367004, 'rouge-2': 0.09564505517482758, 'rouge-3': 0.043253131210803986, 'bleu_score': 0.0, 'exact_match': 100.63999938964844}, 'acceptance_rate': {'mean': -1.0}, 'total_time': {'mean': 1.2253910517692566}, 'time_per_token': {'mean': 0.03829347036778927}, 'tokens_per_second': {'mean': 26.412611989974977}}
        ```

        SS:
        ```console
        $ torchrun benchmark.py --model facebook/layerskip-llama2-7B \
            --dataset xsum_summarization \
            --n_shot 3 \
            --num_samples 100 \
            --generation_strategy self_speculative \
            --num_speculations 12 \
            --exit_layer 8 \
            --output_dir ./logs
        ```
        Result:
        ```console
        {'predicted_text': {'rouge-l': 0.228690505027771, 'rouge-1': 0.28392985463142395, 'rouge-2': 0.09567989408969879, 'rouge-3': 0.043278768658638, 'bleu_score': 0.0, 'exact_match': 100.5999984741211}, 'acceptance_rate': {'mean': 0.43856151334941385}, 'total_time': {'mean': 1.176209304332733}, 'time_per_token': {'mean': 0.03675654076039791}, 'tokens_per_second': {'mean': 28.22171961784363}}
        ```


## Sweep
- Llama 7B continual:
    - Greedy:
        - HumanEval
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset human_eval \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/greedy/human_eval/ \
            --sample False
        ```

        - CNN/DM Summarization
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/greedy/cnn_dm_summarization/ \
            --sample False
        ```

        - XSUM Summarization
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset xsum_summarization \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/greedy/xsum_summarization/ \
            --sample False
        ```

        - CNN/DM Language Modeling
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_lm \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/greedy/cnn_dm_lm/ \
            --sample False
        ```
    - Sampling:
        - HumanEval
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset human_eval \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/sampling/human_eval/ \
            --sample True
        ```

        - CNN/DM Summarization
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_summarization \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/sampling/cnn_dm_summarization/ \
            --sample True
        ```

        - XSUM Summarization
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset xsum_summarization \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/sampling/xsum_summarization/ \
            --sample True
        ```

        - CNN/DM Language Modeling
        ```console
        $ torchrun sweep.py --model facebook/layerskip-llama2-7B \
            --dataset cnn_dm_lm \
            --generation_strategy self_speculative \
            --num_samples 150 \
            --max_steps 256 \
            --output_dir ./logs/llama2_7b/sampling/cnn_dm_lm/ \
            --sample True
        ```


## Correctness
- Llama 7B continual:
    - HumanEval
    ```console
    $ torchrun correctness.py --model facebook/layerskip-llama2-7B \
        --dataset human_eval \
        --generation_strategy self_speculative \
        --num_speculations 6 \
        --exit_layer 4 \
        --num_samples 10 \
        --output_dir ./logs
    ```
    Result:
    ```console
    
    ```

## License
LayerSkip is licensed under CC-by-NC license. Refer to the LICENSE file in the top level directory.

## Contributing
We welcome contributions to LayerSkip. If you are interested in contributing please see [this document](./CONTRIBUTING.md).

## Citation
If you use LayerSkip in your research, please use the following BibTex entry:

```bibtex
@misc{layerskip,
      title={LayerSkip: Enabling Early Exit Inference and Self-Speculative Decoding}, 
      author={Mostafa Elhoushi and Akshat Shrivastava and Diana Liskovich and Basil Hosmer and Bram Wasti and Liangzhen Lai and Anas Mahmoud and Bilge Acun and Saurabh Agarwal and Ahmed Roman and Ahmed A Aly and Beidi Chen and Carole-Jean Wu},
      year={2024},
      booktitle = "Accepted in ACL Main Conference",
      eprint={2404.16710},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```