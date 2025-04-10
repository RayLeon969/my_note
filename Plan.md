```
需软件定制：
620-10-320-01486
660-22-320-01569
620-60-320-00834
620-30-320-01920 反工 有庫存
无需
620-00-320-00013
290-000-0012-02


```

  

 1.数量加入任务卡片  kanban料号换成数量

2. 软件包类型做成选择项 ：正式包：默认    临时包：选择之后路径必填

3. 销售明细绑定的界面默认优先带料号规格中的主客户 如果查不到则带具体客户关联的主客户
4. 绑定的二次确认取消

4. 数量变更同样要把卡片变更为黄色
5. 按钮改为：软件已发文 撤销软件发文





# 软件定制上线流程

itd执行脚本：

```
from odoo.addons.project_extend.script.fix_script import fix_stage_data
fix_stage_data(self.env)

        if flag:
            self.abnormal = True
            Sync = self.env['wms.sync.base']
            for record in self_records:
                if record.default_code in flag:
                    text_line = 'id：{}、料号：{}'.format(record.id, record.default_code if record.default_code else '')
                    result += """<li>{}</li>""".format(text_line)
            result += "<b>------开始尝试修复------</b>"
            for record in self_records:
                if record.default_code in flag:
                    result += f"<b>------开始同步料号：{record.default_code}------</b>"
                    try:
                        Sync.post_to_remote(record)
                        result += f"<li>------同步料号：{record.default_code}成功------</li>"
                    except Exception as e:
                        result += f"<li>------同步料号：{record.default_code}失败，原因：{str(e)}------</li>"
        else:
            result += '<li>无</li>'
```



itd升级project_extend

scm 需要额外升级sys_info

