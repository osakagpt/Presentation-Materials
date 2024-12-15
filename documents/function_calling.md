## GPT系の`structured output`機能とは

開発者が提供するJSONスキーマに適合した構造化出力をモデルが確実に生成する機能
[](https://openai.com/index/introducing-structured-outputs-in-the-api/)

### `response_format`と`tools`
`structured output`には２種類のAPIがある

#### `response_format`

モデルがユーザーに直接構造化された応答を返す場合に使用する。具体的には、モデルは `response_format` で指定された JSON スキーマに適合する JSON オブジェクトを直接生成します。

#### `tools` 

アプリケーションが外部ツールを呼び出してその結果を構造化データとして取得し、ユーザーに返す場合に使用します。モデルは `tools` で定義されたツールの JSON スキーマに従ってツールの実行に必要なパラメータを生成し、ツールの実行結果を構造化データとして受け取る。

### toolsのユースケース

1. データベースクエリの実行: ユーザーの自然言語による質問を SQL クエリに変換し、データベースに対してクエリを実行し、その結果を構造化データとして返すことができる。

2. 外部 API の呼び出し: 天気予報 API や 株価情報 API など、外部の API を呼び出して情報を取得し、その結果を構造化データとして返すことができる。

3. 複雑な計算の実行: 高度な数学的計算や統計処理などを外部のライブラリやサービスに依頼し、その結果を構造化データとして返すことができる。

### ユースケース1: 自然言語のプロンプトから、RDBSにわたすためのSQL文を生成したい

#### Request

```json
POST /v1/chat/completions
{
  "model": "gpt-4o-2024-08-06",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant. The current date is August 6, 2024. You help users query for the data they are looking for by calling the query function."
    },
    {
      "role": "user",
      "content": "look up all my orders in may of last year that were fulfilled but not delivered on time"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "query",
        "description": "Execute a query.",
        "strict": true,
        "parameters": {
          "type": "object",
          "properties": {
            "table_name": {
              "type": "string",
              "enum": ["orders"]
            },
            "columns": {
              "type": "array",
              "items": {
                "type": "string",
                "enum": [
                  "id",
                  "status",
                  "expected_delivery_date",
                  "delivered_at",
                  "shipped_at",
                  "ordered_at",
                  "canceled_at"
                ]
              }
            },
            "conditions": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "column": {
                    "type": "string"
                  },
                  "operator": {
                    "type": "string",
                    "enum": ["=", ">", "<", ">=", "<=", "!="]
                  },
                  "value": {
                    "anyOf": [
                      {
                        "type": "string"
                      },
                      {
                        "type": "number"
                      },
                      {
                        "type": "object",
                        "properties": {
                          "column_name": {
                            "type": "string"
                          }
                        },
                        "required": ["column_name"],
                        "additionalProperties": false
                      }
                    ]
                  }
                },
                "required": ["column", "operator", "value"],
                "additionalProperties": false
              }
            },
            "order_by": {
              "type": "string",
              "enum": ["asc", "desc"]
            }
          },
          "required": ["table_name", "columns", "conditions", "order_by"],
          "additionalProperties": false
        }
      }
    }
  ]
}
```

#### Output JSON
```json
{
  "table_name": "orders",
  "columns": ["id", "status", "expected_delivery_date", "delivered_at"],
  "conditions": [
    {
      "column": "status",
      "operator": "=",
      "value": "fulfilled"
    },
    {
      "column": "ordered_at",
      "operator": ">=",
      "value": "2023-05-01"
    },
    {
      "column": "ordered_at",
      "operator": "<",
      "value": "2023-06-01"
    },
    {
      "column": "delivered_at",
      "operator": ">",
      "value": {
        "column_name": "expected_delivery_date"
      }
    }
  ],
  "order_by": "asc"
}
```

このjsonから一意のSQL文を組み立てることができる

## claudeのtools

cluadeにも`tools`類似機能がある
- [`tool-use`](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)


### ユースケース1: 自然言語のプロンプトから、外部の天気予報APIに渡すためのJSONを出力したい

#### Request
```bash
curl https://api.anthropic.com/v1/messages \
  -H "content-type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "tools": [
      {
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"]
        }
      }
    ],
    "messages": [
      {
        "role": "user",
        "content": "What is the weather like in San Francisco?"
      }
    ]
  }'
```

#### Output JSON
```json
{
  "id": "msg_01234abcd",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01234",
      "name": "get_weather",
      "input": {
        "location": "San Francisco, CA"
      }
    }
  ],
  "tool_calls": [
    {
      "id": "toolu_01234",
      "type": "tool_use",
      "name": "get_weather",
      "input": {
        "location": "San Francisco, CA"
      }
    }
  ],
  "stop_reason": "tool_use",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 85,
    "output_tokens": 20,
    "total_tokens": 105
  }
}
```
