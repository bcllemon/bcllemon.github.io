layout: post
title: "Flask入坑指南"
date: "2016-01-12 00:00"
categories: 技术
---
# Flask
## Session
Flask默认 Session 是基于浏览器 Cookie 的，这样很多从 J2EE 转过来的程序员第一次必然掉坑。Flask 官方给了多种解决方案，具体可以参考 [Sessions](http://flask.pocoo.org/snippets/category/sessions/)。其中基于Redis和 mongodb 的方案比较适合分布式部署。当然你也可以参考里面的实现，按照自己的情况来处理 Session。
## Json返回值
1. FLask出于安全原因不允许返回 Array，只能返回 Object。具体原因参见 [json-security](http://flask.pocoo.org/docs/0.10/security/#json-security)
2. 默认无法序列化 Decimal。
解决办法：安装 simplejson

# Flask-SQLAlchemy/SQLAlchemy
## 同一个事物中 Query 会自动触发 commit
这个问题严格上说不是 Flask 的，实际是SQLAlchemy的。默认 DB 的 Session 是Autoflash的，这样当你在同一个事物中，既有 update 又有 select 操作的时候，就会出现各种灵异错误。解决办法是使用no_autoflush.
```
with db.session.no_autoflush:
    room = Room.query.get(pre_order_info.roomId)
```
## Model无法序列化成 json
model 类继承AutoSerialize，然后转json时调用get_public方法
```
class AutoSerialize(object):
    'Mixin for retrieving public fields of model in json-compatible format'
    __public__ = None

    def get_public(self, exclude=(), extra=()):
        "Returns model's PUBLIC data for jsonify"
        data = {}
        keys = self._sa_instance_state.attrs.items()
        public = self.__public__ + extra if self.__public__ else extra
        for k, field in keys:
            if public and k not in public: continue
            if k in exclude: continue
            value = self._serialize(field.value)
            if value:
                data[k] = value
        return data

    @classmethod
    def _serialize(cls, value, follow_fk=False):
        if isinstance(value, datetime.datetime):
            ret = value.strftime('%Y-%m-%d %H:%M:%S')
        elif isinstance(value, Decimal):
            ret = str(value)
        elif hasattr(value, '__iter__'):
            ret = []
            for v in value:
                ret.append(cls._serialize(v))
        elif AutoSerialize in value.__class__.__bases__:
            ret = value.get_public()
        else:
            ret = value

        return ret
```
# Flask-Auth
许久不更新，与最新版的 Flask 不兼容。这个包本身的设计比较轻巧，此外对于权限的配置是以 Resource+Operation 方式来配置，比较灵活，还是值得一用的。具体修改如下：
# Flask-WTF/WTForms
1. 注意表单类的基类From，要引用flask_wtf中，其在原有WTForms增强了功能。
2. 对于SelectField类型，无法传空值。会导致validate无法通过。
