---
tags:
  - note
  - literature
  - game
  - ffxiv
  - ff14
optional_tags: 
alias: 
duetime: 
created: 2024-06-03
completion: 
share: true
category: entertainment
title: 2024-06-03-fflogs网站api
---

fflogs第二版api采用了 GraphQL API，功能较为强大，其具体的api文档可以在这里找到：
```cardlink
url: https://www.archon.gg/ffxiv/articles/help/api-documentation
title: "API Documentation"
description: "Information on the API documentation of FF Logs."
host: www.archon.gg
favicon: https://assets.rpglogs.com/img/archon/logo.svg
image: https://assets.rpglogs.com/img/ff/featured-article-fallback.jpg
```

# 认证

第二版api采用 OAuth认证，需要注册客户端后获取该次会话的token：

```python
def get_access_token():
# Your Client ID and Client Secret
client_id = secret['client_id']
client_secret = secret['client_secret']

# OAuth Token URL
token_url = 'https://www.fflogs.com/oauth/token'

# Prepare the authentication request
auth_response = requests.post(
    token_url,
    auth=(client_id, client_secret),
    data={'grant_type': 'client_credentials'},
    timeout=10
)

# Extract the access token from the response
access_token = auth_response.json().get('access_token')
return access_token
```

# 数据获取

数据分为公开数据和私有数据，一个report如果被设置为公开，则可以通过[公开数据的api](https://www.fflogs.com/api/v2/client)直接查到，如果未公开则需要通过[私有api](https://www.fflogs.com/api/v2/user)。api的查询方式是一样的，能获取的数据可以通过这里查看：
```cardlink
url: https://www.fflogs.com/v2-api-docs/warcraft/
title: "Graphql schema documentation"
host: www.fflogs.com
```


根据用户名获取绝本相关数据的一个demo，从中可以看到api有速率限制，达到限制后会返回429状态码：

```python
def fetch_graphql_data(
    character_name, group_name, client, start_time=start_time
):
  # 定义GraphQL端点和查询
  url = 'https://www.fflogs.com/api/v2/client'
  headers = {
      'Authorization': f'Bearer {client}',
      'Content-Type': 'application/json'
  }
  query = f"""
    query {{
        characterData {{
            character(
                name: "{character_name}"
                serverSlug: "{group_name}"
                serverRegion: "cn"
            ) {{
                UCoB: encounterRankings(encounterID: 1060)
                UwU: encounterRankings(encounterID: 1061)
                TEA: encounterRankings(encounterID: 1062)
                DSR: encounterRankings(encounterID: 1065)
                TOP: encounterRankings(encounterID: 1068)
            }}
        }}
    }}
    """
  response = requests.post(
      url, json={'query': query}, headers=headers, timeout=10
  )
  for i in range(10):
    if response.status_code == 200:
      break
    if response.status_code == 401:
      # 如果访问令牌过期，重新获取访问令牌并重试
      logging.info("Access token expired, refreshing...")
      client = get_access_token()
      headers['Authorization'] = f'Bearer {client}'
      response = requests.post(
          url, json={'query': query}, headers=headers, timeout=10
      )
    elif response.status_code == 429:
      # 如果达到速率限制，等待一段时间后重试
      logging.info("Rate limit reached, waiting for 1 hour...")
      time_struct = time.localtime(start_time + 3660)

      print("sleep until", time.strftime("%Y-%m-%d %H:%M:%S", time_struct))
      time.sleep((start_time + 3660 - time.time()) % 3660)
      start_time = time.time()
      response = requests.post(
          url, json={'query': query}, headers=headers, timeout=10
      )
  if i == 9:
    raise requests.exceptions.HTTPError(
        f"Failed to fetch data for {character_name} in {group_name}, status code: {response.status_code}"
    )

  return response.json(), client, start_time
```

