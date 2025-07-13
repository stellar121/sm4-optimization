SM4 encryption algorithm implementation and optimization in C++
基于《20250707-sm4-public.pdf》的 SM4 算法软件实现与优化项目项目概述本项目旨在基于《20250707-sm4-public.pdf》文档中的技术细节，实现 SM4 分组密码算法的高效 C++ 版本，并逐步应用文档中提及的优化策略，包括 T-Table 合并变换、SIMD 指令加速及防缓存攻击等技术，最终达成高性能与安全性兼顾的实现目标。算法基础（参考文档第 14-15 页）SM4 是一种采用非平衡 Feistel 结构的分组密码，核心参数如下：分组长度：128 位密钥长度：128 位迭代轮数：32 轮核心变换：轮函数\(F(\cdot)\)、复合变换\(T(\cdot) = L(\tau(\cdot))\)非线性变换\(\tau(\cdot)\)：基于 S 盒的字节替换（文档第 31 页 S 盒定义）线性变换\(L(\cdot)\)：\(B \oplus (B \lll 2) \oplus (B \lll 10) \oplus (B \lll 18) \oplus (B \lll 24)\)代码结构plaintextsm4-optimization/
├── src/                # 实现代码（按优化阶段划分）
│   ├── sm4_base.cpp    # 基础实现（轮函数与密钥扩展，文档第18页）
│   ├── sm4_tbl.cpp     # T-Table优化（合并τ与L变换，文档第21页）
│   ├── sm4_simd.cpp    # SIMD加速（SSE/AVX指令，文档第20-22页）
│   └── sm4_secure.cpp  # 防缓存攻击版本（OpenSSL方案，文档第23-27页）
├── test/               # 测试代码
│   ├── test_vectors.cpp # 标准测试向量验证（文档第31页S盒示例）
│   └── benchmark.cpp    # 性能基准测试（参考文档第41-44页指标）
├── docs/               # 技术文档
│   ├── algo_principle.md  # 算法原理推导（文档第15页轮函数定义）
│   └── optimization.md    # 优化策略说明（文档第21-27页技术细节）
└── CMakeLists.txt      # 编译配置（支持多版本切换）
优化策略（对应文档技术方案）T-Table 方法（文档第 21 页）
预计算 4 个 8×32 位表，合并非线性变换\(\tau(\cdot)\)与线性变换\(L(\cdot)\)，将轮内计算简化为查表操作，消除 4 次循环移位开销，性能从 30.7 cycles/byte 提升至 19.9 cycles/byte。SIMD 指令加速（文档第 20-22 页）
利用 128 位 SSE、256 位 AVX 或 512 位 AVX512 寄存器并行处理多分组，结合PSHUFB指令实现并行 S 盒替换，16 分组时性能可达 9.6 cycles/byte（AVX512 优化）。防缓存攻击（文档第 23-27 页）
参考 OpenSSL 方案，首尾轮使用字节级 S 盒操作，中间轮保留 T-Table，确保内存访问模式与输入无关，平衡性能与抗侧信道攻击能力。编译与使用编译步骤bash# 克隆仓库
git clone https://github.com/stellar121/sm4-optimization.git
cd sm4-optimization

# 构建
mkdir build && cd build
cmake ..
make

# 运行测试
make test
接口示例cpp运行// 初始化密钥
uint8_t key[16] = {0x01, 0x23, 0x45, 0x67, 0x89, 0xab, 0xcd, 0xef, 
                   0xfe, 0xdc, 0xba, 0x98, 0x76, 0x54, 0x32, 0x10};

// 基础加密
SM4Base sm4(key);
uint8_t plaintext[16] = {0x01, 0x23, 0x45, 0x67, 0x89, 0xab, 0xcd, 0xef,
                         0xfe, 0xdc, 0xba, 0x98, 0x76, 0x54, 0x32, 0x10};
uint8_t ciphertext[16];
sm4.encrypt(plaintext, ciphertext);
性能指标（参考文档第 41-44 页）实现版本1 分组（cycles/byte）16 分组（cycles/byte）基础实现30.76.5T-Table 优化19.94.8AVX512 加速15.50.66防缓存攻击版本17.87.5参考资料技术文档：《20250707-sm4-public.pdf》标准规范：GM/T 0002-2012、GB/T 32907-2016
