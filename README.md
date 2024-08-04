# claudette-pydantic


<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

> Adds Pydantic support for
> [claudette](https://github.com/AnswerDotAI/claudette) through function
> calling

claudette_pydantic provides the `struct` method in the `Client` and
`Chat` of claudette

`struct` provides a wrapper around `__call__`. Provide a

## Install

``` sh
pip install claudette_pydantic
```

## Getting Started

``` python
from claudette.core import *
import claudette_pydantic
from pydantic import BaseModel, Field, ValidationError
from typing import Literal, Union, List, Optional
```

``` python
model = models[-1]
model
```

    'claude-3-haiku-20240307'

``` python
class Pet(BaseModel):
    "Create a new pet"
    name: str
    age: int
    owner: str = Field(default="NA", description="Owner name. Do not return if not given.")
    type: Literal['dog', 'cat', 'mouse']

c = Client(model)
print(repr(c.struct(msgs="Can you make a pet for my dog Mac? He's 14 years old", resp_model=Pet)))
print(repr(c.struct(msgs="Tom: my cat is juma and he's 16 years old", resp_model=Pet)))
```

    Pet(name='Mac', age=14, owner='NA', type='dog')
    Pet(name='juma', age=16, owner='Tom', type='cat')

We can go way deeper, for example this one I pulled from [pydantic
docs](https://docs.pydantic.dev/latest/concepts/unions/#discriminated-unions)

``` python
class Cat(BaseModel):
    pet_type: Literal['cat']
    meows: int


class Dog(BaseModel):
    pet_type: Literal['dog']
    barks: float


class Reptile(BaseModel):
    pet_type: Literal['lizard', 'dragon']
    scales: bool


class OwnersPets(BaseModel):
    """
    Information for to gather for an Owner's pets
    """
    pet: List[Union[Cat, Dog, Reptile]] = Field(..., discriminator='pet_type')

chat = Chat(model)
pr = "hello I am a new owner and I would like to add some pets for me. I have a dog which has 6 barks, a dragon with no scales, and a cat with 2 meows"
chat.struct(OwnersPets, pr=pr)
```

    OwnersPets(pet=[Dog(pet_type='dog', barks=6.0), Reptile(pet_type='dragon', scales=False), Cat(pet_type='cat', meows=2)])

While the struct uses tool use to enforce the schema, we save the tool
use as json to keep the user,assistant,user flow.

``` python
chat.h
```

    [{'role': 'user',
      'content': [{'type': 'text',
        'text': 'hello I am a new owner and I would like to add some pets for me. I have a dog which has 6 barks, a dragon with no scales, and a cat with 2 meows'}]},
     {'role': 'assistant',
      'content': [{'type': 'text',
        'text': "OwnersPets(pet=[Dog(pet_type='dog', barks=6.0), Reptile(pet_type='dragon', scales=False), Cat(pet_type='cat', meows=2)])"}]}]

Continue the conversation…

``` python
chat.struct(OwnersPets, pr="actually my dragon does have scales, can you change that for me?")
```

    OwnersPets(pet=[Dog(pet_type='dog', barks=6.0), Reptile(pet_type='dragon', scales=True), Cat(pet_type='cat', meows=2)])

Alternatively you can use the original tool use flow with
`treat_as_output=False` (but requires the next input to be assistant)

``` python
chat.struct(OwnersPets, pr=pr, treat_as_output=False)
chat.h[-3:]
```

    [{'role': 'user',
      'content': [{'type': 'text',
        'text': 'hello I am a new owner and I would like to add some pets for me. I have a dog which has 6 barks, a dragon with no scales, and a cat with 2 meows'}]},
     {'role': 'assistant',
      'content': [ToolUseBlock(id='toolu_015Kyxycj43DBej69qRZ8jaL', input={'pet': [{'pet_type': 'dog', 'barks': 6}, {'pet_type': 'dragon', 'scales': False}, {'pet_type': 'cat', 'meows': 2}]}, name='OwnersPets', type='tool_use')]},
     {'role': 'user',
      'content': [{'type': 'tool_result',
        'tool_use_id': 'toolu_015Kyxycj43DBej69qRZ8jaL',
        'content': "pet=[Dog(pet_type='dog', barks=6.0), Reptile(pet_type='dragon', scales=False), Cat(pet_type='cat', meows=2)]"}]}]

You can even add few shot examples *for each input*

``` python
class User(BaseModel):
    "User creation tool"
    age: int = Field(description='Age of the user')
    name: str = Field(title='Username')
    password: str = Field(
        json_schema_extra={
            'title': 'Password',
            'description': 'Password of the user',
            'examples': ['Monkey!123'],
        }
    )
res = c.struct(msgs=["Can you create me a new user for tom age 22"], resp_model=User, sp="for a given user, generate a similar password based on examples")
print(res)
```

    age=22 name='tom' password='Monkey!123'

### Signature:

``` python
Client.struct(
    self: claudette.core.Client,
    msgs: list,
    resp_model: type[BaseModel], # non-initialized Pydantic BaseModel
    **, # Client.__call__ kwargs...
) -> BaseModel
```

``` python
Chat.struct(
    self: claudette.core.Chat,
    resp_model: type[BaseModel], # non-initialized Pydantic BaseModel
    treat_as_output=True, # In chat history, tool is reflected
    **, # Chat.__call__ kwargs...
) -> BaseModel
```
