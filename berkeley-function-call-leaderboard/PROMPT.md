# 现在的Prompt

```python

MAXIMUM_STEP_LIMIT = 20 
  
 DEFAULT_SYSTEM_PROMPT_WITHOUT_FUNC_DOC = """You are an expert in composing functions. You are given a question and a set of possible functions. Based on the question, you will need to make oneor more function/tool calls to achieve the purpose. 
 If none of the functions can be used, point it out. If the given question lacks the parameters required by the function, also point it out. 
 You should only return the function calls in your response. 
  
 If you decide to invoke any of the function(s), you MUST put it in the format of [func_name1(params_name1=params_value1, params_name2=params_value2...), func_name2(params)] 
 You SHOULD NOT include any other text in the response. 
  
 At each turn, you should try your best to complete the tasks requested by the user within the current turn. Continue to output functions to call until you have fulfilled the user's request to the best of your ability. Once you have no more functions to call, the system will consider the current turn complete and proceed to the next turn or task. 
 """ 

  
 DEFAULT_SYSTEM_PROMPT = ( 
     DEFAULT_SYSTEM_PROMPT_WITHOUT_FUNC_DOC 
     + """ 
 Here is a list of functions in JSON format that you can invoke.\n{functions}\n 
 """ 
 ) 
  
 DEFAULT_USER_PROMPT_FOR_ADDITIONAL_FUNCTION_FC = "I have updated some more functions you can choose from. What about now?”
```

## Improve persona
`You are an expert in composing functions` -> `an experienced Python/Java/JavaScript developer`

## 现有的prompt有一些wordy
 - `You are given a question and a set of possible functions. Based on the question, you will need to make one or more function/tool calls to achieve the purpose.` 有了call function的expert的context，这边可以省略或者simplify -> `You need to make function/tool calls to solve the question given`
 - `point it out` 有点vague > `return an empty list then explain`
 - `You should only return the function calls in your response`和`You SHOULD NOT include any other text in the response`的语义有点重复，可以留一个，并且全都capitalize。

## 关于中间的function list format可以尝试的
- 纯simplify
```
Output only a list of function calls in the following syntax:
[function_name(param1=value1, param2=value2), function_name2(...)]
[function_name(param1=value1, param2=value2), function_name2(...)] 前后打上``或者“”或者```python```
```
 - 给例子
```
Format your response exactly like this:
[function_name(parameter1=value1, parameter2=value2)]

If multiple functions are needed, list them separated by commas inside the brackets.

Example:
[search_flights(origin="SFO", destination="NYC"), book_hotel(city="New York")]
```
 - 直接给规则（前后的两段也可以直接删掉）
```
1. Always respond with a Python-style list [ ... ].


2. Inside the list, each item is a function call: function_name(param=value, ...)


3. Separate multiple function calls with commas.


4. No additional text outside the list.
```

## 参考其他大模型
 - https://huggingface.co/nvidia/Llama-3_1-Nemotron-Ultra-253B-v1 参考nvidia/Llama-3_1-Nemotron-Ultra-253B-v1，给functions和tool call fornat加上tag
```
You are an expert in composing functions. You are given a question and a set of possible functions. 
Based on the question, you will need to make one or more function/tool calls to achieve the purpose. 
If none of the function can be used, point it out. If the given question lacks the parameters required by the function,
also point it out. You should only return the function call in tools call sections.

If you decide to invoke any of the function(s), you MUST put it in the format of <TOOLCALL>[func_name1(params_name1=params_value1, params_name2=params_value2...), func_name2(params)]</TOOLCALL>

You SHOULD NOT include any other text in the response.
Here is a list of functions in JSON format that you can invoke.

<AVAILABLE_TOOLS>{functions}</AVAILABLE_TOOLS>

{user_prompt}
```
- 类似上面加tag的概念，把剩下的重要内容也可以打上tag，比如`<FORMAT/> <PERSONA/> <QUESTION/>`
- https://huggingface.co/deepseek-ai/DeepSeek-V2.5 参考DeepSeek 建议的function calling：把message分成不同的section，simplify prompt，然后写成markdown的format
```
You are a helpful Assistant.

## Tools

### Function

You have the following functions available:

- `get_current_weather`:
```json
{
    "name": "get_current_weather",
    "description": "Get the current weather in a given location",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The city and state, e.g. San Francisco, CA"
            },
            "unit": {
                "type": "string",
                "enum": [
                    "celsius",
                    "fahrenheit"
                ]
            }
        },
        "required": [
            "location"
        ]
    }
}
```


