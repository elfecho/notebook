## 调取事件

需求：使用jsx调用elementui的事件时，需要使用驼峰法，并且前面需要加on

例如：

RFC写法：
```javascript
<el-table
    data={data}
    row-click={handleClick}
>
</el-table>
```
jsx写法：
```javascript
<el-table
    data={data}
    onRowClick={handleClick}
>
</el-table>
```

## slot插槽使用

需求：自定义table单元格的内容, 使用v-slots调用表行内传递的值

RFX写法：

```vue
<el-table :data="tableData">
    <el-table-column label="NANE">
        <template slot-scope="scope">  
            <p>{{scope.row.name}}</p>
        </template>
</el-table>
```

JSX写法：

```jsx
<el-table :data="tableData">
    <el-table-column 
        label="NANE"
        prop="name"
        v-slots={{
            default: ({ row, column, $index }) => row.name
        }}
    ></el-table-column>
        
</el-table>
```

