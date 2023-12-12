---
title: 'GatsbyのsiteMetadataにカスタムデータを追加する'
date: '2023-04-17T12:25:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Gatsby
tags:
  - Gatsby
---
# GatsbyのsiteMetadataにカスタムデータを追加する

- Node.js 18.16.0
- Gatsby 5.8.1

## gatsby-config.ts

```typescript
const config: GatsbyConfig = {
  siteMetadata: {
    siteUrl: "https://example.com",
    title: "Example Blog",
    myCustomData: myCustomData,
  },
  graphqlTypegen: true,
  // ...
}
```

## gatsby-node.ts

```typescript
export const createSchemaCustomization: GatsbyNode['createSchemaCustomization'] = ({ actions }) => {
  const { createTypes } = actions

  createTypes(`
    type MyCustomData {
      text: String
      flag: Boolean
    }
    type SiteSiteMetadata {
      myCustomData: MyCustomData
    }
  `)
}
```

## mypage.tsx

```typescript
import * as React from "react"
import {
  graphql,
  PageProps,
} from 'gatsby'
import { GetMyPageQuery } from '../gatsby-types'

const MyPage: React.FC<PageProps<GetMyPageQuery>> = (props) => {
  const data = props.data

  const site = data?.site
  const myCustomData = site?.siteMetadata?.myCustomData
  // ...
}

export const pageQuery = graphql`
  query GetMyPage {
    site {
      siteMetadata {
        myCustomData {
          text
          flag
        }
      }
    }
  }
`

export default MyPage
```

## 参考

- [https://github.com/gatsbyjs/gatsby/issues/1781](https://github.com/gatsbyjs/gatsby/issues/1781)
- [https://stackoverflow.com/questions/61530280/unable-to-filter-custom-data-in-sitemetadata-in-gatsby-using-graphql-in-graphiql](https://stackoverflow.com/questions/61530280/unable-to-filter-custom-data-in-sitemetadata-in-gatsby-using-graphql-in-graphiql)
- [https://tomiko0404.hatenablog.com/entry/2022/02/20/gatasby-graphql-datalayer](https://tomiko0404.hatenablog.com/entry/2022/02/20/gatasby-graphql-datalayer)
- [https://stackoverflow.com/questions/62984585/gatsby-how-to-handle-undefined-fields-in-sitemetadata](https://stackoverflow.com/questions/62984585/gatsby-how-to-handle-undefined-fields-in-sitemetadata)
