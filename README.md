# wings-sanic

Welcome to wings-sanic README file.

The wings-sanic is a lightweight python framework aimed at making it as simple as possible to document your Sanic API with Swagger UI, Plus param validation and model serialization.


## What is wings-sanic
The wings-sanic(required `Python3.5+`) is lightweight python framework for [sainc](https://sanic.readthedocs.io/). This framework provides:

- declarative generation of http handler interfaces by parsing function annotations
- validation and serialization to and from a json.
- validation and serialization to and from native python objects.
- validation and serialization from complex object.
- autodocumentation of all handlers generated by this way, via [swagger](https://swagger.io/).
-  publish and handle event(message)


## Installation
```bash
$ pip install wings-sanic
```
if you want publish or handle event(message) and use the default [mq-server(rabbitmq)](./wings_sanic/mq_servers/rabbitmq_server.py),
you need install third pkg [aioamqp](https://github.com/polyconseil/aioamqp)

## Quick Start
### Overview:

![Image text](./docs/simple.png)

### Simple example

```python
import wings_sanic
from wings_sanic import application, settings, event, utils

# -----------  user settings -------------
dev_settings = {
    'DEFAULT_CONTEXT': {
        'response_shape': 'wings_sanic.views.ResponseShapeCodeDataMsg'
    },

    'MQ_SERVERS': {
        'default': {
            'server': 'wings_sanic.mq_servers.rabbitmq_server.MqServer',
            'url': 'amqp://guest:guest@127.0.0.1:5672/test',
            'exchange': 'test',
            'reconnect_delay': 5.0
        }
    },

    'SWAGGER': {
        'info': {
            "version": '1.0.0',
            "title": 'Authors API'
        },
        'schemes': ['http']
    },
    'HTTP_PORT': 8080,
    'DEBUG': True,
    'DEV': True,
    'CORS': True
}
settings.load(**dev_settings)

authors_db = {
    1: {'name': '张三', 'phone': '17788661234', 'id': 1, 'password': '123456'}
}


def get_author_id():
    author_id = getattr(get_author_id, 'author_id', 1) + 1
    setattr(get_author_id, 'author_id', author_id)
    return author_id


class Author(wings_sanic.Serializer):
    id = wings_sanic.IntField('Id', read_only=True)
    name = wings_sanic.StringField("姓名", required=True)
    phone = wings_sanic.PhoneField('电话', required=True)
    password = wings_sanic.StringField('密码', required=True, write_only=True)


authors = wings_sanic.WingsBluePrint('authors', url_prefix='api/authors')


@authors.get('/', response_serializer=Author(many=True))
async def list_authors(request, *args, **kwargs):
    """
    authors列表
    返回authors列表，不分页
    """
    return authors_db.values()


@authors.post('/', body_serializer=Author(), response_serializer=Author())
async def create_author(request, body, *args, **kwargs):
    """
    新建author
    """
    author_id = get_author_id()
    body['id'] = author_id
    authors_db[author_id] = body
    await event.publish(event.DomainEvent('AuthorCreated', **body))
    return body


@authors.get('/<author_id>/',
             path_params={'author_id': wings_sanic.IntField('作者Id')},
             response_serializer=Author())
async def author_detail(request, author_id, *args, **kwargs):
    """
    获取指定id的author详情
    """
    return authors_db.get(author_id, None)


@event.handler('AuthorCreated')
def handle_author_created(evt):
    print(utils.to_primitive(evt))


# --------------------- main -----------------
if __name__ == '__main__':
    application.app.blueprint(authors)
    application.start()


```

Run and then you can get swagger documents by browser `http//:0.0.0.0:8080/swagger/`

### Sample Project
You can start a project from [template_main](./wings_sanic/template_main.py)

More, a [sample project](./examples/sample) prepared for you.


## Document
Todo
