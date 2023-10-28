---
layout: post
author: savio
title: JSON in ActiveRecord
categories: ['database', 'rails']
tags: ['db', 'mysql', 'json']
image:
  path: /assets/img/posts/json-logo.png
  alt: JSON
---

For example we have next migration:
```
class CreateRecord < ActiveRecord::Migration[7.0]
  def change
    create_table :records do |t|
      t.json :data

      t.timestamps
    end
  end
end

```

and `data` has value like
```json
  {"id"=>"bd732784-9583-493e-a55e-64fcc85317bd",
    "mail"=>"user@user.com",
    "country"=>"Ukraine",
    "memberOf"=>[{"@odata.type"=>"#microsoft.graph.group", 
                  "displayName"=>"All Company"},
                 {"@odata.type"=>"#microsoft.graph.group", 
                   "displayName"=>"develop"}],
    "businessPhones"=>["555-767-7676"]}
```

Select
```
Record.select("data->>'$.mail' as mail")
```

Filtering:
```
Record.where("data->>'$.mail' = ?", 'user@user.com'))
Record.where("LOWER(data->>'$.mail') LIKE ?", '%@user.com'))

```

