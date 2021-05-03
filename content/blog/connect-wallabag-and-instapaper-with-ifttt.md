---
title: "Connect Wallabag and Instapaper with IFTTT"
date: 2018-09-11T19:20:48+08:00
---

I have migrated all my readings from Instapaper and Pocket to Wallabag since Sep-2018, those 2 awesome services have been my primary reading apps during the past years, they are elegant and reliable. But what makes me to host my own reading app are I want to own my data, extend the services as I want and search in full articles. To avoid to put all eggs in one basket, I decided still use Instapaper as the first line of saving articles, besides it has a lot of awesome features(e.g. Sending to Kindle) and best integration with other internet services.

Wallabag has a [api method](https://doc.wallabag.org/en/developer/api/oauth.html) to submit url itself, but the problem is the access token expires in 1 hour. There is already an api wrapper for fixing this problem, [wallabag_api by push-things](https://github.com/push-things/wallabag_api), it handles the auth gracefully. So next you just need to write your RESTful api to consume it, [flask-restful](https://flask-restful.readthedocs.io/en/latest/) is my first choice.

The first step: install wallabag_api, flask, flask-restful.

```sh
pip install wallabag_api
pip install flask-restful
```

Get you own `clent_id` and `client_srec` in the **API clients management** page of your Wallabag website and replace with below script.

```py
### wapi.py
import aiohttp
import asyncio
from pathlib import Path
from wallabag_api.wallabag import Wallabag

from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

parser = reqparse.RequestParser()
parser.add_argument('token')
parser.add_argument('url')
parser.add_argument('content')

# settings
my_host = 'http://my-wallabag-host:33333'

async def main(loop, url):

    params = {'username': 'username',
              'password': 'password',
              'client_id': '8888888888',
              'client_secret': '9999999999',
              'extension': 'json'}

    # get a new token
    token = await Wallabag.get_token(host=my_host, **params)

    # initializing
    async with aiohttp.ClientSession(loop=loop) as session:
        wall = Wallabag(host=my_host,
                        client_secret=params.get('client_secret'),
                        client_id=params.get('client_id'),
                        token=token,
                        extension=params['extension'],
                        aio_sess=session)
        await wall.post_entries(url)


class Entry(Resource):

    def post(self):
        args = parser.parse_args()
        if args['token'] == "my-own-static-token":
            loop.run_until_complete(main(loop, args['url']))


api.add_resource(Entry, '/entry')
loop = asyncio.get_event_loop()
```

For running you little RESTful service you'd better choose a management app like [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/). Run below command will start you service at port 55555 and no logs saving, you can change the output direction into you preferred logs location.

```sh
nohup uwsgi --http :55555 --module wapi:app >/dev/null 2>&1 &
```

After you comfortable with your testing of the service, you can config at IFTTT like mine:

{{% center %}}
    {{< figure src="../img/ifttt-config-wallabag-instapaper.jpg" width="100%" alt="">}}
{{% /center %}}

## Other thoughts

I have had other ideas and really like to try in the future:

### Not real time sync

Using IFTTT to save the article links first and deploy a task to import unsynced articles weekly, the temp links can be saved in Google Sheet, Excel, Airtable or even DynamoDB.

### Run the api in AWS Lambda

Since AWS is very generous to the developer, you can deploy this service for personal use at no charge(or nearly free)
