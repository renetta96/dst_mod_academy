# V. Nâng cao, tips & tricks

## 1. Bọc hàm (wrap function)

Trong phần [Bắt Đầu](getting_started.md), mình đã giới thiệu cách bọc hàm để thay đổi chức năng của 1 entity. Phương thức này chỉ có thể làm được khi bạn muốn xen logic mới vào trước hoặc sau hàm được bọc. Trong ví dụ ở [Bắt Đầu](getting_started.md), rất dễ để bọc hàm `onnear` vì chúng ta chỉ cần thêm 1 câu lệnh `if` rồi `return` ở trước hàm được bọc.

Tuy nhiên, việc bọc hàm không phải lúc nào cũng dễ dàng như vậy. Ở ví dụ này, chúng ta sẽ mod sao cho player lấy honey từ beebox không bị ong bay ra dí. Cũng như ví dụ trước, đầu tiên cần kiểm tra code của beebox. Prefab của beebox là `beebox`, nằm ở `prefabs/beebox.lua` trong code DST. Hàm `onharvest` chính là hàm được gọi khi player thu hoạch honey:

```
local function onharvest(inst, picker, produce)
    --print(inst, "onharvest")
    if not inst:HasTag("burnt") then
		if produce == levels[1].amount then
			AwardPlayerAchievement("honey_harvester", picker)
		end
        updatelevel(inst)
        if inst.components.childspawner ~= nil and not TheWorld.state.iswinter then
            inst.components.childspawner:ReleaseAllChildren(picker)
        end
    end
end
```

Ở đây, `inst` chính là beebox, `picker` thường là player (không chắc Bearger có lấy được mật ong không), `produce` là số mật ong lấy được. Nếu beebox không bị cháy rụi, và lượng mật lấy được là tối đa, player sẽ được achievement (cái này không chắc trong DST có không). Sau đó, gọi hàm `updatelevel` để điều chỉnh lại hoạt ảnh của beebox theo số lượng mật đang có (ở đây là còn 0). Tiếp theo, nếu không phải mùa đông, gọi ong ra dí `picker`.

Hàm này không thể bọc được, đơn giản vì không có cách nào ngăn nó gọi `inst.components.childspawner:ReleaseAllChildren` chỉ bằng cách xen logic vào trước vào sau. Nếu bạn xen vào trước lệnh `if` kiểm tra picker là player thì return, bạn sẽ khiến hàm `updatelevel` không được gọi, player không được achievement. Cũng không có cách nào điều khiển `ReleaseAllChildren` không gọi với `picker` cả. Còn nếu bạn xen logic vào sau, thì `ReleaseAllChildren` đã được gọi rồi. Nói cách khác, trong trường hợp hàm `onharvest` này, ta buộc phải chen logic vào giữa hàm. Đương nhiên bạn có thể copy nguyên hàm này ra, điều chỉnh nó và gán lại vào component `harvestable` của beebox. Nhưng như đã nói, cách này không nên làm vì dễ gây conflict và không thể tự động cập nhật khi game update.

Để giải quyết vấn đề này, ta cần wrap chính hàm `ReleaseAllChildren` của component `childspawner` của beebox. Ý tưởng đó là thêm logic vào trước hàm này, nếu picker là player thì return. Hàm này được định nghĩa trong `components/childspawner.lua`:

```
function ChildSpawner:ReleaseAllChildren(target, prefab)
	local failures = 0 -- prevent infinate loops when SpawnChild fails to spawn its child
	while self:CanSpawn() and failures < 3 do
		failures = self:SpawnChild(target, prefab) == nil and (failures + 1) or 0
	end

	failures = 0
	self:UpdateMaxEmergencyCommit()
	while self:CanEmergencySpawn() and failures < 3 do
		failures = self:SpawnEmergencyChild(target, prefab) == nil and (failures + 1) or 0
	end
end
```

`ChildSpawner` là 1 class, cú pháp `function ChildSpawner:ReleaseAllChildren` tức là định nghĩa method `ReleaseAllChildren` của class `ChildSpawner`. Bên trong method của class, bạn có thể sử dụng biến đặc biệt `self` chính là object của class đó. (`target`, `prefab`) là tham số của method. Như ở hàm `onharvest`, `inst.components.childspawner:ReleaseAllChildren(picker)` được gọi, tức `target` chính là `picker`, còn `prefab` sẽ là `nil` (rỗng, không có gì), còn `self` chính là `inst.components.childspawner`. `method` thực chất cũng chỉ là 1 function bình thường, cú pháp

```
inst.components.childspawner:ReleaseAllChildren(picker)
```

thực chất tương đương với

```
inst.components.childspawner.ReleaseAllChildren(inst.components.childspawner, picker)
```

Method `ReleaseAllChildren` thực chất là 1 hàm nhận vào 3 tham số `(childspawnerObject, picker, prefab)`, nhưng khi viết bằng cú pháp `childspawnerObject:ReleaseAllChildren`, `childspawnerObject` sẽ tự động được truyền vào là tham số đầu tiên, do đó chỉ cần truyền vào từ tham số thứ 2 trở đi. Sau đó bên trong method, `self` chính là tham số đầu tiên này, hay chính là `childspawnerObject`.

Để bọc hàm này, chúng ta làm như sau:

```
local function BeeBoxPostInit(prefab)
  if prefab.components.childspawner then
    local oldReleaseAllChildren = prefab.components.childspawner.ReleaseAllChildren
    prefab.components.childspawner.ReleaseAllChildren = function(comp, target, ...)
      if target and target:HasTag("player") then
        return
      end

      oldReleaseAllChildren(comp, target, ...)
    end
  end
end

AddPrefabPostInit("beebox", BeeBoxPostInit)
```

Cú pháp `...` đại diện cho các tham số truyền vào sau đó. Vì ở đây chúng ta chỉ quan tâm 2 tham số đầu tiên là `childspawnerObject` (hay `comp`), và `target`, nên có thể sử dụng `...` để đại diện cho những tham số ở sau mà chúng ta không cần động đến. Sau đó, chèn logic kiểm tra `target` là player thì return. Nếu không phải player, gọi hàm `oldReleaseAllChildren`, chính là hàm được bọc.