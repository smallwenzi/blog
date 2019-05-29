title: 底层代码构造sql忽略NUll条件
author: wen
tags:
  - jpa
categories:
  - spring cloud
date: 2019-05-06 11:00:00
---
代码没有判断查询的值null，导致把全部数据load内存里，这样系统崩溃

代码：com.amway.common.jpa.repository.JpaRepoUtil
```

	private static StringBuilder generateQlForQueryByProperties(Map<String, Object> propertiesMap,
			StringBuilder ql, Map<Integer, Object> params, FindByPropertiesType findType) {
		int index = 0;
		
		if (propertiesMap != null && !propertiesMap.isEmpty()) {
			boolean doDeleteAnd = false, doDeletewWere = true;
			ql.append(" where ");
	        for (Entry<String,Object> entry: propertiesMap.entrySet()) {
	        	Object value = entry.getValue();
                //忽略null 的值
	        	if (value != null) {
	        		if (doDeletewWere) 
	        			doDeletewWere = false;
	        		
	        		if (value instanceof String && (findType == null || FindByPropertiesType.Fuzzy.equals(findType))) {
	        			String valueStr = (String) entry.getValue();
	        			ql.append("upper(model." + entry.getKey() + ") like :propertyValue" + index + " and ");
	        			params.put(index++, "%" + valueStr.trim().toUpperCase() + "%");
	        		} else if(QueryValue.NULL.equals(value)) {
	        			ql.append("model." + entry.getKey() + " is null and ");
	        		} else if(QueryValue.NotNull.equals(value)) {
	        			ql.append("model." + entry.getKey() + " is not null and ");
	        		} else {
	        			ql.append("model." + entry.getKey() + "=:propertyValue" + index + " and ");
	        			params.put(index++, entry.getValue());
	        		}
	        		
	        		doDeleteAnd = true;
	        	}
	        }
	        if (doDeleteAnd)
	        	ql = ql.delete(ql.toString().lastIndexOf(" and "), ql.toString().length());
	        if (doDeletewWere)
        		ql = ql.delete(ql.toString().lastIndexOf(" where "), ql.toString().length());
		}
		return ql;
	}
```

![upload successful](A问题/1.png)

总结：需要深度了解底层代码。