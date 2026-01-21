.. _metadata:

#################
合约的元数据
#################

.. index:: metadata, contract verification

Solidity编译器会自动生成一个JSON文件。该文件包含关于编译合约的两种信息：

- 如何与合约进行交互：ABI和NatSpec文档。
- 如何重现编译并验证已部署的合约：编译器版本，编译器设置和使用的源文件。
  
编译器默认会将元数据文件的IPFS哈希附加到每个合约的运行字节码（不一定是创建字节码）的末尾，
这样，如果发布了合约，您可以以经过身份验证的方式检索该文件，而无需依赖于集中式数据提供者。
其他可用选项包括Swarm哈希和不将元数据哈希附加到字节码中。
这些选项可以通过 :ref:`标准JSON接口<compiler-api>` 的配置进行设置。

您必须将元数据文件发布到IPFS，Swarm或其他服务，
以便其他人可以访问它。您可以通过使用 ``solc --metadata`` 命令
和 ``--output-dir`` 参数来创建该文件。如果没有这个参数，
元数据将被写到标准输出。
元数据包含 IPFS 和 Swarm 对源代码的引用，
所以除了元数据文件外，您还必须上传所有的源文件。
对于IPFS， ``ipfs add`` 返回的 CID 中包含的哈希值（不是文件的直接sha2-256哈希值）
应与字节码中包含的哈希值相匹配。

元数据文件的格式如下所示。下面的例子是以人类可读的方式呈现的。
正确格式化的元数据应正确地使用引号，
将空格减少到最小，并按字母顺序对所有对象的键进行排序，
以达到规范化的格式。是不允许有注释的，这里的目的只是为了解释。

.. code-block:: javascript

    {
      // 必选：编译器的详情，内容视语言而定。
      "compiler": {
        // 可选：生成此输出的编译器二进制文件的哈希值。
        "keccak256": "0x123...",
        // 对 Solidity 语言来说是必选的：编译器的版本
        "version": "0.8.2+commit.661d1103"
      },
      // 必选：源代码语言，基本上是选择规范中的一个“子版本”。
      "language": "Solidity",
      // 必选：关于合约生成的信息。
      "output": {
        // 必选：合约的ABI定义。参见“合约ABI规范”。
        "abi": [/* ... */],
        // 必选：合约的NatSpec开发者文档。请参阅 https://docs.soliditylang.org/en/latest/natspec-format.html 获取详细信息。
        "devdoc": {
          // 合约中 @author NatSpec字段的内容
          "author": "John Doe",
          // 合约中 @dev NatSpec字段的内容
          "details": "Interface of the ERC20 standard as defined in the EIP. See https://eips.ethereum.org/EIPS/eip-20 for details",
          "errors": {
            "MintToZeroAddress()" : {
              "details": "Cannot mint to zero address"
            }
          },
          "events": {
            "Transfer(address,address,uint256)": {
              "details": "Emitted when `value` tokens are moved from one account (`from`) toanother (`to`).",
              "params": {
                "from": "The sender address",
                "to": "The receiver address",
                "value": "The token amount"
              }
            }
          },
          "kind": "dev",
          "methods": {
            "transfer(address,uint256)": {
              // @dev NatSpec字段的内容
              "details": "Returns a boolean value indicating whether the operation succeeded. Must be called by the token holder address",
              // @param NatSpec字段的内容
              "params": {
                "_value": "The amount tokens to be transferred",
                "_to": "The receiver address"
              },
              // @return NatSpec字段的内容
              "returns": {
                // 如果存在，返回var名称（这里是 “success”）。如果返回的var是未命名的，“_0” 作为键。
                "success": "a boolean value indicating whether the operation succeeded"
              }
            }
          },
          "stateVariables": {
            "owner": {
              // 状态变量的 @dev NatSpec 字段的内容
              "details": "Must be set during contract creation. Can then only be changed by the owner"
            }
          },
          // 合约中 @title NatSpec字段的内容
          "title": "MyERC20: an example ERC20",
          "version": 1 // NatSpec 版本
        },
        // 必选：合约的NatSpec用户文档。请参阅”NatSpec格式“
        "userdoc": {
          "errors": {
            "ApprovalCallerNotOwnerNorApproved()": [
              {
                "notice": "The caller must own the token or be an approved operator."
              }
            ]
          },
          "events": {
            "Transfer(address,address,uint256)": {
              "notice": "`_value` tokens have been moved from `from` to `to`"
            }
          },
          "kind": "user",
          "methods": {
            "transfer(address,uint256)": {
              "notice": "Transfers `_value` tokens to address `_to`"
            }
          },
          "version": 1 // NatSpec版本
        }
      },
      // 必选：编译器设置。反映了编译过程中JSON输入中的设置。
      // 请查阅标准JSON输入的文档中的“settings”字段
      "settings": {
        // Solidity所需的内容：元数据所创建的文件路径和合约或库的名称。
        "compilationTarget": {
          "myDirectory/myFile.sol": "MyContract"
        },
        // Solidity所需的内容。
        "evmVersion": "london",
        // Solidity所需的内容：所使用的库合约的地址。
        "libraries": {
          "MyLib": "0x123123..."
        },
        "metadata": {
          // 反映了输入JSON中使用的设置，默认为“true”
          "appendCBOR": true,
          // 反映了输入JSON中使用的设置，默认为“ipfs”
          "bytecodeHash": "ipfs",
          // 反映了输入JSON中使用的设置，默认为“false”
          "useLiteralContent": true
        },
        // 可选：优化器设置。字段“enabled”和“runs”已被弃用，仅用于向后兼容。
        "optimizer": {
          "details": {
            "constantOptimizer": false,
            "cse": false,
            "deduplicate": false,
            // inliner的默认值为“false”
            "inliner": false,
            // jumpdestRemover的默认值为“true”
            "jumpdestRemover": true,
            "orderLiterals": false,
            // peephole的默认值为“true”
            "peephole": true,
            "yul": true,
            // 可选：仅在“yul”为“true”时出现
            "yulDetails": {
              "optimizerSteps": "dhfoDgvulfnTUtnIf...",
              "stackAllocation": false
            }
          },
          "enabled": true,
          "runs": 500
        },
        // Solidity所需的内容：按顺序排列的导入重映射列表。
        "remappings": [ ":g=/dir" ]
      },
      // 必选：编译源文件/源单元，键为文件路径
      "sources": {
<<<<<<< HEAD
        "destructible": {
          // 必选（除非使用“url”）：源文件的字面内容
          "content": "contract destructible is owned { function destroy() { if (msg.sender == owner) selfdestruct(owner); } }",
          // 必选：源文件的keccak256哈希值
=======
        "settable": {
          // Required (unless "url" is used): literal contents of the source file
          "content": "contract settable is owned { uint256 private x = 0; function set(uint256 _x) public { if (msg.sender == owner) x = _x; } }",
          // Required: keccak256 hash of the source file
>>>>>>> v0.8.25
          "keccak256": "0x234..."
        },
        "myDirectory/myFile.sol": {
          // 必选：源文件的keccak256哈希值
          "keccak256": "0x123...",
          // 可选：源文件中提供的SPDX许可证标识符
          "license": "MIT",
          // 必选（除非使用“content”，参见上文）：指向源文件的按顺序排列的URL（或URLs），
          // 协议可以是任意的，但建议使用IPFS URL
          "urls": [ "bzz-raw://7d7a...", "dweb:/ipfs/QmN..." ]
        }
      },
      // 必选：元数据格式的版本
      "version": 1
    }

.. warning::
  由于产生的合约的字节码默认包含元数据哈希值，
  对元数据的任何改变都可能导致字节码的改变。
  这包括对文件名或路径的改变，而且由于元数据包括所有使用的源的哈希值，
  一个空白的改变就会导致不同的元数据和不同的字节码。

.. note::
    上面的ABI定义没有固定的顺序。它可以随着编译器的版本而改变。
    不过，从Solidity 0.5.12版本开始，该数组保持一定的顺序。

.. _encoding-of-the-metadata-hash-in-the-bytecode:

在字节码中对元数据哈希值进行编码
=============================================

编译器目前默认将
IPFS 哈希值（在 CID v0 中）<https://docs.ipfs.tech/concepts/content-addressing/#version-0-v0>`_ 
的规范元数据文件和编译器版本附加到字节码的末尾。
也可以使用 Swarm 哈希值代替 IPFS，或使用实验标志。
以下是所有可能的字段：

.. code-block:: javascript

    {
      "ipfs": "<metadata hash>",
      // 如果编译器设置中的 “bytecodeHash” 是 “bzzr1”，那就没有使用 “ipfs”，而是 “bzzr1”
      "bzzr1": "<metadata hash>",
      // 以前的版本使用的是 “bzzr0” 而不是 “bzzr1”
      "bzzr0": "<metadata hash>",
      // 如果使用了任何影响代码生成的实验功能
      "experimental": true,
      "solc": "<compiler version>"
    }

由于我们将来可能会支持以其他方式检索元数据文件，
因此这些信息被存储为 `CBOR <https://tools.ietf.org/html/rfc7049>`_-编码。
字节码中的最后两个字节表示 CBOR 编码信息的长度。通过观察这个长度，
可以用 CBOR 解码器对字节码的相关部分进行解码。

请查看 `元数据游乐场（Metadata Playground） <https://playground.sourcify.dev/>`_ 以了解其运行情况。

solc的发布版本使用如上所示的3个字节的版本编码
（主要、次要和补丁版本号各一个字节），
而预发布版本将使用一个完整的版本字符串，包括提交哈希和构建日期。

命令行标志 ``--no-cbor-metadata`` 可以用来跳过元数据在部署的字节码末端的附加。
同样地，标准JSON输入中的布尔字段 ``settings.metadata.appendCBOR`` 可以设置为false。

.. note::
  CBOR映射也可能包含其他键，
  因此最好通过查看字节码末尾的CBOR长度来完全解码数据，
  并使用适当的CBOR分析器。不要依赖以 ``0xa264``
  或 ``0xa2 0x64 'i' 'p' 'f' 's'`` 开头的数据。

自动化接口生成和NatSpec 的使用方法
====================================================

元数据的使用方式如下：一个想要与合约交互的组件
（例如钱包）会检索合约的代码。
它对包含元数据文件的 IPFS/Swarm 哈希的 CBOR 编码部分进行解码。
通过该哈希值，元数据文件被检索出来。该文件被 JSON 解码成一个类似于上述的结构。

然后，该组件可以使用ABI为合约自动生成一个基本的用户界面。

此外，钱包可以使用 NatSpec 用户文档，每当用户与合约交互时，
就会向用户显示一条可读的确认信息，同时请求对交易签名进行授权。

有关其他信息，请阅读 :doc:`以太坊自然语言规范（NatSpec）格式 <natspec-format>`。

源代码验证的用法
==================================

如果已固定/发布，则可以从 IPFS/Swarm 获取合约的元数据。
元数据文件还包含源文件的URLs或IPFS哈希值，
以及编译设置，即重现编译所需的一切信息。

有了这些信息，就可以通过重现编译过程来验证合约的源代码，
并将编译的字节码与已部署合约的字节码进行比较。


由于源代码的哈希值是元数据的一部分，因此也会自动验证源代码。
文件或设置的任何变化都会导致不同的元数据哈希值。
元数据是整个编译过程的指纹。

`Sourcify <https://sourcify.dev>`_ 利用这一功能进行 “完全/完美验证”，
并将文件公开固定在IPFS上，以便使用元数据哈希值进行访问。
