我现在来完整陈述一下我的问题，现在有如下代码：

```
class DataSyncBase(models.AbstractModel):
    _name = 'data.sync.base'
    _description = '同步基础'
    
    @api.model
    def data_sync_remote_create(self, vals_list):
        return self.try_sync_method('_data_sync_remote_create', vals_list)

    @api.model
    def _data_sync_remote_create(self, vals_list):
        new_vals_list = []
        print(len(vals_list))
        for val in vals_list:
            val = self._trans_sync_fields(val)
            print(val)
            exist = self.search([('unique_id', '=', val['unique_id'])])
            if 'id' in val and val['id']:
                exist = self.browse(val['id'])
                del val['id']
            if not exist:
                new_vals_list.append(val)
            else:
                exist.write(val)
        if new_vals_list:
            res = self.create(new_vals_list)
            self.data_sync_operation_after_create(res)
            
    def _trans_sync_fields(self, vals):
        def _convert_foreign_key_fields(data):
            value_type = data['type']
            if value_type in ['many2one', 'many2many']:
                target_objs = self.env.get(data['model_name']).search([('unique_id', 'in', data['unique_ids'])])
                if value_type == 'many2one':
                    return target_objs.id
                elif value_type == 'many2many':
                    return [(6, 0, target_objs.ids)]

        self._data_sync_check_after(vals)

        if 'id' in vals:
            del_flag = True
            if vals['id']:
                static_record = self.env['ir.model.data'].search(
                    [('model', '=', self._name), ('name', '=', vals['id'])])
                if static_record:
                    vals['id'] = static_record.res_id
                    del_flag = False
            if del_flag:
                del vals['id']

        for each_fields, each_value in vals.items():
            if isinstance(each_value, dict):
                value_type = each_value['type']
                if value_type in ['many2one', 'many2many']:
                    vals[each_fields] = _convert_foreign_key_fields(each_value)
                elif value_type == 'many2many_create':
                    for create_data in each_value['data']:
                        for each_create_fields, each_create_value in create_data[2].items():
                            if isinstance(each_create_value, dict):
                                value_type = each_create_value['type']
                                if value_type in ['many2one', 'many2many']:
                                    create_data[2][each_create_fields] = _convert_foreign_key_fields(each_value)
        return vals
        
        
        
class MrpBomLine(models.Model):
    _name = 'mrp.bom.line'
    _inherit = ['mrp.bom.line', 'data.sync.base']
    
    def _trans_sync_fields(self, vals):
        vals = super()._trans_sync_fields(vals)
        product_product = self.env['product.product']
        if 'product_id' in vals:
            product = product_product.search([('product_tmpl_id.unique_id', '=', vals['product_id'])])
            if product:
                vals['product_id'] = product.id
            else:
                vals['product_id'] = False
        return vals

```

现在我再同步mrp_bom_line的时候传入了28条数据

即_data_sync_remote_create方法的vals_list的print(len(vals_list))显示为28

但是for val in vals_list:内部，print(val)的数量我数了一下只有26条。我debug发现那两条本应该是要被放入new_vals_list然后创建的，但是循环执行到26条后结束了，这是不对的，我想知道怎么解决。

我现在给出vals_list的结构:

```
vals_list = [
{'unique_id': 'mrp.bom.line:f1f70a2d-e610-49ca-8d4c-8356bb42464b', 'bom_id': 27, 'product_id': 403, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '7183KADC B2.8mm N制机型选用', 'sequence': 1, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:8ca1e936-99ea-4483-977a-b913f89bc91f', 'bom_id': 27, 'product_id': 404, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 2, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:68b20ddc-16a6-4f62-8acd-a73367643f76', 'bom_id': 27, 'product_id': 405, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '机身内', 'sequence': 3, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:c2ce521d-5899-4ef9-8fef-f0cd66f12f32', 'bom_id': 27, 'product_id': 406, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '机身内', 'sequence': 4, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:72b09eb3-8168-403d-83ef-2146f2d83866', 'bom_id': 27, 'product_id': 407, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 5, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:2cb95a53-ad8e-49e9-80d8-9bf5e92bcb9e', 'bom_id': 27, 'product_id': 408, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 6, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:7b186d5d-ce3c-47ff-b2ac-7f66003b9708', 'bom_id': 27, 'product_id': 409, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 7, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:4430c9b9-24cc-48d8-b69f-2bea1e5414aa', 'bom_id': 27, 'product_id': 410, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 8, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:68fd61dd-a4d9-47c9-8e04-349daf26fb93', 'bom_id': 27, 'product_id': 114, 'make_up_qty': 12.0, 'main_qty': 1.0, 'material_tag': False, 'note': '固定灯板4、固定主板4、固定铁柱4', 'sequence': 9, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:4222d90b-43e4-49d5-b38d-5c7f27006eb2', 'bom_id': 27, 'product_id': 411, 'make_up_qty': 4.0, 'main_qty': 1.0, 'material_tag': False, 'note': '锁前盖', 'sequence': 10, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:798d3748-9ca3-4d55-b21d-234c5f0e2f55', 'bom_id': 27, 'product_id': 412, 'make_up_qty': 4.0, 'main_qty': 1.0, 'material_tag': False, 'note': '支撑', 'sequence': 11, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:60c97e6a-c98e-4f8f-aaa1-2cf3d8aea818', 'bom_id': 27, 'product_id': 413, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '主板JP16连接红外灯板', 'sequence': 12, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:7e48c790-cc16-422a-a869-fae98e254aff', 'bom_id': 27, 'product_id': 414, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '主板JP17连接红外灯板', 'sequence': 13, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:e61692dd-378a-44ed-a33f-cef64d239c50', 'bom_id': 27, 'product_id': 415, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '7183KADC机型选用', 'sequence': 14, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:7d1b6fe1-4449-4fae-8843-7f732d100713', 'bom_id': 27, 'product_id': 416, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '配件', 'sequence': 15, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:3ec1d772-fdbe-4f4e-950e-5485a2b20836', 'bom_id': 27, 'product_id': 95, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '装摄像机', 'sequence': 16, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:c010d1bc-8b3c-4a07-b9b3-f65b6b2d670c', 'bom_id': 27, 'product_id': 417, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 17, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:5cd67e53-a8f5-4a6c-9232-cb6c8fa5ca52', 'bom_id': 27, 'product_id': 89, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 18, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:a16aeda7-cb55-436a-8394-e69d3eab1c4c', 'bom_id': 27, 'product_id': 418, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '机身标', 'sequence': 19, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:503c296d-06cf-497b-86d6-9641d424d817', 'bom_id': 27, 'product_id': 85, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '包装盒标', 'sequence': 20, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:7c944d33-6e96-4063-b2cf-5c4e38b53f81', 'bom_id': 27, 'product_id': 420, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 21, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:b9ec6043-a48e-4110-bc85-100a8b21a118', 'bom_id': 27, 'product_id': 421, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': 'B2.8mm镜头选用', 'sequence': 22, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:a956cf85-d612-4394-80ae-c728201ef376', 'bom_id': 27, 'product_id': 423, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 23, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:6833b86a-e74a-4ff7-8a5a-53d3c066b8cb', 'bom_id': 27, 'product_id': 424, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': False, 'sequence': 24, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:58b25c8e-1c62-48a5-905f-58f74a08bca1', 'bom_id': 27, 'product_id': 419, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '贴在多头组合线上， 标贴距离线头距离4cm', 'sequence': 25, 'ending_date': False, 'replace_type_id': 1}
{'unique_id': 'mrp.bom.line:038e5ce9-edb7-4850-af19-c6c92e93d98f', 'bom_id': 27, 'product_id': 422, 'make_up_qty': 1.0, 'main_qty': 1.0, 'material_tag': False, 'note': '配件，配件包与网络接口保护盖贴二维码标', 'sequence': 26, 'ending_date': False, 'replace_type_id': 2}
]
```

