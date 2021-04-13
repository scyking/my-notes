# MyBatis 动态 sql

标签（空格分隔）： mybatis mysql

---

[toc]

## xml 标签

### `if`

```
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### `choose（when、otherwise）`
> 从多个条件中选择一个使用。

```
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### `trim（where、set）`

- 动态添加`where`，会自动去除子句开头`AND\OR`。

    ```
    <select id="findActiveBlogLike"
         resultType="Blog">
      SELECT * FROM BLOG
      <where>
        <if test="state != null">
             state = #{state}
        </if>
        <if test="title != null">
            AND title like #{title}
        </if>
        <if test="author != null and author.name != null">
            AND author_name like #{author.name}
        </if>
      </where>
    </select>
    ```
    
    等价：
    
    ```
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
        ...
    </trim>
    ```

- 动态更新

    ```
    <update id="updateAuthorIfNecessary">
      update Author
        <set>
          <if test="username != null">username=#{username},</if>
          <if test="password != null">password=#{password},</if>
          <if test="email != null">email=#{email},</if>
          <if test="bio != null">bio=#{bio}</if>
        </set>
      where id=#{id}
    </update>
    ```
    
    等价：
    
    ```
    <trim prefix="SET" suffixOverrides=",">
      ...
    </trim>
    ```

### `foreach`
> 对集合进行遍历。

```
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

### `bind`
> 创建一个变量，并将其绑定到当前的上下文。

```
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```


## 接口类上

### `script`

```
@Update({"<script>",
  "update Author",
  "  <set>",
  "    <if test='username != null'>username=#{username},</if>",
  "    <if test='password != null'>password=#{password},</if>",
  "    <if test='email != null'>email=#{email},</if>",
  "    <if test='bio != null'>bio=#{bio}</if>",
  "  </set>",
  "where id=#{id}",
  "</script>"})
void updateAuthorValues(Author author);
```