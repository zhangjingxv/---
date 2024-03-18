模型下载链接https://wxkk6cr9s4x.feishu.cn/drive/folder/OPf4fqS2xlN7HddB4ZxclydQn8f
书生-金融大模型

finance-chat
license evaluation

🤗HuggingFace | OpenXLab_Model | ModelScope

OpenXLab_App | 🆕Update News | 🤔Reporting Issues 丨 bilibili

English | 简体中文

📝目录
📖 简介
🚀 News
🛠️ 使用方法
快速开始
重新训练
环境搭建
XTuner微调
OpenXLab应用部署
LMDeploy量化
OpenCompass评测
LMDeploy & OpenCompass量化以及量化评测
💕 致谢
开源许可证
📖 简介
AM (Advanced Mathematics) chat 是一个集成了金融知识及其解答的大语言模型。该模型使用 Finance 解析融合的数据集，基于 InternLM2-Finance-7B 模型，通过 xtuner 微调，专门设计用于解答金融问题。

如果你觉得这个项目对你有帮助，欢迎 ⭐ Star，让更多的人发现它！

🛠️ 使用方法
快速开始
下载模型
从 ModelScope
从 OpenXLab
本地部署
git clone https://github.com/zhangjingxv/shusheng.git
python start.py
Docker部署
docker run -t -i --rm --gpus all -p 8501:8501 guidonsdocker/amchat:latest bash start.sh
重新训练
环境搭建
clone 本项目
git clone https://github.com/zhangjingxv/shusheng.git
cd AMchat
创建虚拟环境
conda env create -f environment.yml
conda activate AMchat
pip install -r requirements-raw.txt
XTuner微调
准备配置文件

列出所有内置配置
xtuner list-cfg

mkdir -p /root/finance/data
mkdir /root/finance/config && cd /root/finance/config

xtuner copy-cfg internlm2_chat_7b_qlora_oasst1_e3 .
模型下载
mkdir -p /root/finance/model
download.py

import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('Shanghai_AI_Laboratory/internlm2-finance-7b', cache_dir='/root/finance/model')
修改配置文件
cd /root/finance/config
vim internlm_chat_7b_qlora_oasst1_e3_copy.py

修改模型为本地路径
pretrained_model_name_or_path = 'internlm/internlm-chat-7b'
pretrained_model_name_or_path = './internlm2-finance-7b'
修改训练数据集为本地路径
data_path = 'timdettmers/openassistant-guanaco'
data_path = './data'
开始微调
xtuner train /root/finance/config2/internlm2_chat_7b_qlora_oasst1_e3_copy.py
PTH 模型转换为 HuggingFace 模型
xtuner convert pth_to_hf ./internlm2_chat_7b_qlora_oasst1_e3_copy.py
./work_dirs/internlm2_chat_7b_qlora_oasst1_e3_copy/epoch_3.pth
./hf
HuggingFace 模型合并到大语言模型
export MKL_SERVICE_FORCE_INTEL=1
export MKL_THREADING_LAYER='GNU'
原始模型参数存放的位置
export NAME_OR_PATH_TO_LLM=/root/finance/model/Shanghai_AI_Laboratory/internlm2-finance-7b

Hugging Face格式参数存放的位置
export NAME_OR_PATH_TO_ADAPTER=/root/finance/config/hf

最终Merge后的参数存放的位置
mkdir /root/finance/config/work_dirs/hf_merge
export SAVE_PATH=/root/finance/config/work_dirs/hf_merge

执行参数Merge
xtuner convert merge
$NAME_OR_PATH_TO_LLM
$NAME_OR_PATH_TO_ADAPTER
$SAVE_PATH
--max-shard-size 2GB
Demo
streamlit run web_demo.py --server.address=0.0.0.0 --server.port 7860
OpenXLab应用部署
仅需要 Fork 本仓库，然后在 OpenXLab 上创建一个新的项目，将 Fork 的仓库与新建的项目关联，即可在 OpenXLab 上部署 AMchat。

Demo

AMchat 与 InternLM2-Finance-7B 在积分问题上对于同一问题的解答。 AMchat 回答正确，InternLM2-Finance-7B 回答错误。
Demo Demo

LMDeploy量化
首先安装LMDeploy
pip install -U lmdeploy
然后转换模型为turbomind格式
--dst-path: 可以指定转换后的模型存储位置。

lmdeploy convert internlm2-chat-7b 要转化的模型地址 --dst-path 转换后的模型地址
LMDeploy Chat 对话
lmdeploy chat turbomind 转换后的turbomind模型地址
OpenCompass评测
安装 OpenCompass
git clone https://github.com/open-compass/opencompass
cd opencompass
pip install -e .
下载解压数据集
cp /share/temp/datasets/OpenCompassData-core-20231110.zip /root/opencompass/
unzip OpenCompassData-core-20231110.zip
评测启动！
python run.py
--datasets finance_gen
--hf-path 模型地址
--tokenizer-path tokenizer地址
--tokenizer-kwargs padding_side='left' truncation='left' trust_remote_code=True
--model-kwargs device_map='auto' trust_remote_code=True
--max-seq-len 2048
--max-out-len 16
--batch-size 2
--num-gpus 1
--debug
LMDeploy & OpenCompass量化以及量化评测
W4 量化评测
KV Cache 量化评测
结果文件与评测数据集可在同目录文件results中获取
