# LLMアプリケーションの本番環境について
LLMアプリケーションの最大の特性はローカルLLMのメモリ容量。ひとつのモデルインスタンスを稼働しているだけで数十GBのメモリを消費する。

もうひとつの特性は、通常のウェブアプリよりも応答に圧倒的な時間がかかる。応答を素早くさせる必要があるならCPUではなくGPUリソース必須。


# AWS EC2 オンデマンドインスタンス
[](https://aws.amazon.com/jp/ec2/pricing/on-demand/)
- p2.xlarge インスタンス(4vCPU, 61GiB)
    - 米国東部リージョン: 0.9$/h→月額92010円
    - 東京リージョン：1.55$/h→月額158472円
- x2gd.xlarge インスタンス(4vCPU, 64GiB)
    - 米国東部リージョン: 0.35$/h→月額35784円


# LLMファインチューニング用GPU
ファインチューニングでは24時間フル稼働する必要はないが、学習には強力なGPUが必須
- google ColabのA100インスタンスが最良の選択肢ぽい
- google colaboの無料版はRAM 12.68GB、strage: 107.72GBでRAMが足りなくなる
- google colabo pro だとA100だと83.5GB、V100では51.5GB使える

