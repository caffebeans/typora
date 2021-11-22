### 协查函

**associate**

```
id ObjectId 主键id
number String 协查函编号
create_org_id String 创建者组织机构id
create_user_id String 创建用户id
create_user_nickname String 创建用户姓名
disease_id String 协查函对应传染病病种id
disease_name String 协查函对应传染病病种名称
approval_user_id String 审核用户id
approval_user_nickname String 审核用户名称
associate_org_id String 协查单位机构id
status int 协查函状态(0-草稿，1-未审核，2-未发布，3-已驳回，4-已发布)
associate_content:{ 协查函内容
	title String 标题
	receiver String 协查单位名称
	content  String 内容
	contacts String 联系人
	phone String 联系电话
	fax  String 传真
	email String 邮箱
	create_org_name String, //创建者组织机构名称
    time String //时间(yyyy-mm-dd)
}
information:[ 协查信息表
	{
		name     姓名
		phone	 电话
		id_card	 身份证
		address	 地址
	}
]
records:[ 处理记录
	{
		user_id
		name 处理人
		opt 操作类型(新建，审核通过，审核驳回，编辑，发布)
		opinion 处理意见
		time 时间
	}
]
createtime
updatetime
```



### 协查函-模板

**associate_temp**

```
id ObjectId 主键id
create_org_id String 创建者组织机构id
create_org  String 创建者组织机构
create_user_id String 创建用户id
create_user_nickname String 创建用户姓名
disease_id String 协查函对应传染病病种id
disease_name String 协查函对应传染病病种名称
associate_content:{ 协查函内容
	title String 标题
	content  String 内容
}
createtime
updatetime
```



### 信息通报

**information_notice**

```
id ObjectId 主键id
create_org_id String 创建者组织机构id
create_org  String 创建者组织机构
create_user_id String 创建用户id
create_user_nickname String 创建用户姓名
disease_id String 协查函对应传染病病种id
disease_name String 协查函对应传染病病种名称
approval_user_id Array 审核用户id
information_notice_content:{ 信息通报内容
	title    标题
	name     姓名
	phone	 电话
	id_card	 身份证
	address	 地址
	。。。。
}
status int 协查函状态(0-草稿，1-未审核，2-通过，3-驳回)
createtime
updatetime
```



### 解除告知单

**remove_notice**

```
id ObjectId 主键id
create_org_id String 创建者组织机构id
create_org  String 创建者组织机构
create_user_id String 创建用户id
create_user_nickname String 创建用户姓名
disease_id String 协查函对应传染病病种id
disease_name String 协查函对应传染病病种名称
approval_user_id Array 审核用户id
remove_notice_content:{ 信息通报内容
	title		标题
	name     	姓名
	id_type  	证件类型
	id_number	身份证
	start_time  隔离开始时间
	end_time	隔离结束时间
	isolation_type 隔离类型(1-家里，2-集中)
	time		时间
}
status int 协查函状态(0-草稿，1-未审核，2-通过，3-驳回)
createtime
updatetime
```



### 互认说明

**mutual_explain**

```
id ObjectId 主键id
create_org_id String 创建者组织机构id
create_org  String 创建者组织机构
create_user_id String 创建用户id
create_user_nickname String 创建用户姓名
disease_id String 协查函对应传染病病种id
disease_name String 协查函对应传染病病种名称
approval_user_id Array 审核用户id
approval_user_nickname String 审核用户名称
mutual_explain_content:{ 信息通报内容
	title		标题
	content     内容
}
information:[ 联络人信息表
	{
		name     姓名
		phone	 电话
		id_card	 身份证
		address	 地址
	}
]
status int 协查函状态(0-草稿，1-未审核，2-通过，3-驳回)
createtime
updatetime
```

