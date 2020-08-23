# IV. Bắt đầu

Hãy bắt đầu với 1 mod đơn giản: cho phép Wilson chạy qua **Killer Bee Hive** mà không bị ong dí.

## 1. Chuẩn bị


Đầu tiên, bạn vào thư mục `Steam/steamapps/common/Don't Starve Together/data/databundles/`, copy `scrips.zip` ra đâu đó, giải nén và kéo vào Text editor của bạn. Đây chính là game logic code của DST, bạn sẽ phải đọc code này **RẤT NHIỀU** để có thể viết mod.

Tiếp theo, download thư mục <https://github.com/renetta96/dst_mod_academy/tree/master/resources/mod_template_simple>, đây là một template mod đơn giản, vừa đủ cho mod này.

## 2. Sơ lược

Về thư mục `mod_template_simple`, cơ bản chỉ có 2 file sau:

- `modinfo.lua`: Đây đơn giản chỉ là file chứa các thông tin về mod của bạn. Bạn có thể sửa `name` - tên mod, `description` - mô tả mod, `author` - tên hoặc nickname bạn, `version` - phiên bản hiện tại của mod.
- `modmain.lua`: Đây là file main của mod, nơi DST sẽ đọc và thực thi mọi lệnh trong này. Hiện tại file `modmain.lua` gần như không có gì cả, đây sẽ là nơi bạn sẽ viết code cho mod này.

Sơ qua một chút về code game của DST, nó sử dụng [Entity component system](https://en.wikipedia.org/wiki/Entity_component_system). Tóm gọn lại, mọi thứ trong game đều là 1 `entity`, đơn thuần chỉ là 1 cục dữ liệu, không hơn không kém. Thứ định nghĩa hành vi của 1 `entity` là các `components`, được "cắm" vào `entity`. Ví dụ, là 1 con người, bạn có thể có các components `talker` (có thể nói), `locomotor` (có thể di chuyển), `sleeper` (có thể ngủ), v.v Một bụi berry sẽ không có `locomotor` vì nó không thể di chuyển, nhưng sẽ có `harvestable` (có thể thu hoạch).

Giờ hãy nhìn vào thư mục giải nén code của DST, có một số thư mục sau:
- `components`: chính là thư mục chứa code của các components.
- `prefabs`: thư mục chứa code của các `entity`.
- `brains`: thư mục chứa code điều khiển hành vi của các `entity`, ví dụ như nhện, lợn, v.v Bạn không cần đụng vào nó trong mod này.
- `modutil.lua`: file chứa các hàm hữu dụng cho việc mod, bạn sẽ phải dùng 1 trong các hàm này.

## 3. Thực hành

Việc đầu tiên khi làm mod thay đổi tính năng của 1 entity đó là đọc code của entity đó. Trước tiên, bạn cần biết tên prefab của Killer Bee Hive. Bạn có thể search google ra DST wiki: <https://dontstarve.fandom.com/wiki/Beehive>, tên prefab của Killer Bee Hive là `wasphive`. Code của `wasphive` có thể được tìm thấy ở `prefabs/wasphive.lua` trong thư mục code DST. Hãy mở file này lên.

Kéo xuống dưới cùng của file, bạn có thể thấy:

```
return Prefab("wasphive", fn, assets, prefabs)
```

Dòng này có nghĩa là: tạo 1 loại entity mới, tên là `wasphive`, sử dụng hàm tên `fn` để khởi tạo, sử dụng các `assets` (ví dụ như animation, ảnh, âm thanh, v.v.) và sử dụng các `prefabs` - có nghĩa là code của entity này phụ thuộc vào 1 vài entities khác, trong trường hợp này là `killerbee`, bởi vì `wasphive` sẽ spawn ra `killerbee`.

OK, tiếp tục kiểm tra hàm `fn`, mình sẽ giải thích sơ qua từng đoạn code trong hàm này:

```
local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddMiniMapEntity()
    inst.entity:AddNetwork()

    MakeObstaclePhysics(inst, 0.5)

    inst.MiniMapEntity:SetIcon("wasphive.png") --replace with wasp version if there is one.

    inst.AnimState:SetBank("wasphive")
    inst.AnimState:SetBuild("wasphive")
    inst.AnimState:PlayAnimation("cocoon_small", true)

    inst:AddTag("structure")
    inst:AddTag("hive")
    inst:AddTag("WORM_DANGER")

    MakeSnowCoveredPristine(inst)

    inst.entity:SetPristine()
```

Đoạn trên đại loại là: tạo ra 1 entity, lưu vào biến `inst`. Cho phép entity này có hình dạng - vị trí vật lý, có hoạt ảnh, có thể phát ra âm thanh, có biểu tượng trên mininap, có thể hoạt động qua network (liên quan đến server - client). Sau đó, định nghĩa entity này sử dụng animation build nào, animation mặc định là `cocoon_small`, thêm 1 vài tags. Đoạn này bạn không cần động vào.

```
    if not TheWorld.ismastersim then
        return inst
    end
```

Dòng `if` này chính là đoạn phân chia logic giữa server và client, mình có nhắc đến trong [Sơ lược](overview.md). `if not TheWorld.ismastersim` có nghĩa là nếu cái máy đang chạy đoạn code này không phải là server, hay nói cách khác là client, thì trả lại `inst` ngay lập tức. Code từ sau `if` này trở đi sẽ chỉ được thực thi trên server.

```
    -------------------------
    inst:AddComponent("health")
    inst.components.health:SetMaxHealth(250) --increase health?
```

Component đầu tiên: `health`. Component này có chức năng cho phép 1 entity có `máu`, do đó `wasphive` có 250 máu.

```
    -------------------------
    inst:AddComponent("childspawner")
    --Set spawner to wasp. Change tuning values to wasp values.
    inst.components.childspawner.childspawner = "killerbee"
    inst.components.childspawner:SetMaxChildren(TUNING.WASPHIVE_WASPS)
    inst.components.childspawner.emergencychildname = "killerbee"
    inst.components.childspawner.emergencychildrenperplayer = 1
    inst.components.childspawner:SetMaxEmergencyChildren(TUNING.WASPHIVE_EMERGENCY_WASPS)
    inst.components.childspawner:SetEmergencyRadius(TUNING.WASPHIVE_EMERGENCY_RADIUS)
```

Component `childspawner`, đây là component phụ trách việc cho phép `wasphive` gọi ra các `killerbee`, tấn công người chơi.

```
    -------------------------
    inst:AddComponent("lootdropper")
    inst.components.lootdropper:SetLoot({ "honey", "honey", "honey", "honeycomb" })
```

Component `lootdropper`, cho phép `wasphive` khi bị phá hủy rơi ra phần thưởng, mà ở đây là 3 honeys, 1 honeycomb.

```
    -------------------------
    MakeLargeBurnable(inst)
    inst.components.burnable:SetOnIgniteFn(OnIgnite)
    -------------------------
```

Hàm `MakeLargeBurnable` được định nghĩa trong `standardcomponents.lua` trong code DST, nó thêm vào component `burnable`, có thể cháy được. Khi bắt đầu bốc cháy, gọi hàm `OnIgnite`.

```
    inst:AddComponent("playerprox")
    inst.components.playerprox:SetDist(10, 13) --set specific values
    inst.components.playerprox:SetOnPlayerNear(onnear)
    inst.components.playerprox:SetPlayerAliveMode(inst.components.playerprox.AliveModes.AliveOnly)
```

Component `playerprox` chịu trách nhiệm theo dõi khi có player đến gần và thực thi hành động nào đó. Trong trường hợp này, khi có player đến gần, hàm `onnear` sẽ được chạy.

```
    -------------------------
    inst:AddComponent("combat")
    --wasp hive should trigger on proximity, release wasps.
    inst.components.combat:SetOnHit(onhitbyplayer)
    inst:ListenForEvent("death", OnKilled)
    -------------------------
```

Component `combat` cho phép bạn có thể đập nó. Khi bị đập, hàm `onhitbyplayer` sẽ được chạy. Khi bị đập chết, hàm `OnKilled` sẽ được chạy.

```
    MakeMediumPropagator(inst)
    MakeSnowCovered(inst)
    -------------------------
    inst:AddComponent("inspectable")
    inst.OnEntitySleep = OnEntitySleep
    inst.OnEntityWake = OnEntityWake

    inst:AddComponent("hauntable")
    inst.components.hauntable:SetHauntValue(TUNING.HAUNT_MEDIUM)
    inst.components.hauntable:SetOnHauntFn(OnHaunt)

    return inst
end
```

Đoạn cuối này, `MakeSnowCovered` cho phép `wasphive` có thể bị tuyết che phủ vào mùa đông, component `inspectable` cho phép bạn có thể inspect `wasphive`. Component `hauntable` cho phép khi bạn chết, trở thành ma, có thể ám nó. Sau cùng, nó trả lại `inst`, là biến lưu entity.

Mỗi khi một tổ ong cần được tạo ra, hàm `fn` sẽ được chay, trả về `inst` là một entity, đã được định nghĩa đầy đủ các tính năng. Việc bạn cần làm bây giờ là thay đổi các tính năng này để khi có Wilson chạy lại gần, `wasphive` không thả `killerbee` ra đánh. Do đó hiển nhiên, chúng ta phải tác động vào component `playerprox` và hàm `onnear`.

Đây là hàm `onnear`:

```
local function onnear(inst, target)
    --hive pop open? Maybe rustle to indicate danger?
    --more and more come out the closer you get to the nest?
    if inst.components.childspawner ~= nil then
        inst.components.childspawner:ReleaseAllChildren(target, "killerbee")
    end
end
```

`target` ở đây chính là player đang lại gần, còn `inst` chính là `wasphive`. Hàm này đơn giản là, khi 1 player lại gần, sử dụng component `childspawner`, gọi tất cả `killerbee` ra dí vào mặt bạn. Vậy là chúng ta đi đúng hướng, vì đúng là trong game, khi bạn lại gần `wasphive`, cả đàn `killerbee` sẽ bay ra đuổi theo bạn.

Trong ngôn ngữ lập trình, một hàm khi đã được định nghĩa, rất khó có thể thay đổi nó. Do đó chúng ta không thể can thiệp trực tiếp vào hàm `onnear`, vì nó đã được định nghĩa rồi. Vậy làm sao để có thể khiến nó làm điều ta cần làm? Hãy kiểm tra cách hàm `onnear` được sử dụng:

```
inst.components.playerprox:SetOnPlayerNear(onnear)
```

Code của component `playerprox` nằm ở `components/playerprox.lua` trong code DST. Có thể hiểu, mỗi entity có một property đặc biệt tên `components`. `components` là 1 dictionary, lưu dữ liệu dưới dạng key => value. Trong trường hợp này, `inst.components.playerprox` là một object của class `PlayerProx`. Giờ kiểm tra hàm `SetOnPlayerNear`:

```
function PlayerProx:SetOnPlayerNear(fn)
    self.onnear = fn
end
```

`self` ở đây chính là object `PlayerProx`, hay chính là object `inst.components.playerprox`. Như vậy hàm `onnear` định nghĩa trong file `wasphive.lua` được gán vào property `onnear` của `inst.components.playerprox`. Chúng ta sẽ can thiệp vào property này.

`modutil.lua` trong code DST cung cấp hàm `AddPrefabPostInit` dùng để thay đổi một entity sau khi nó được tạo ra. Hàm này nhận vào 2 tham số: `prefab` - tên của prefab mà hàm sẽ thay đổi, `fn` - một function nhận vào entity sau khi được tạo ra. Hàm này sẽ thực thi trên cả server và client.

Bạn hãy thêm đoạn code sau vào `modmain.lua`:

```
local function WaspHivePostInit(inst)
  if inst.components.playerprox then
    local oldOnNearFn = inst.components.playerprox.onnear
    local onnearfn = function(inst, target)
      if target and target.prefab == 'wilson' then
        return
      end

      if oldOnNearFn ~= nil then
        oldOnNearFn(inst, target)
      end
    end
    inst.components.playerprox:SetOnPlayerNear(onnearfn)
  end
end

AddPrefabPostInit("wasphive", WaspHivePostInit)
```

Hàm này hoạt động như sau:

- `if inst.components.playerprox then`: vì hàm này sẽ được chạy trên cả server và client, trong khi mình chỉ muốn nó chạy trên server, mình có thể viết như này. Dòng `if` này kiểm tra xem `inst` có component `playerprox` hay không. Chỉ có server mới thêm component `playerprox` cho `wasphive`, do đó đoạn code này chỉ chạy trên server. Mặt khác, việc kiểm tra component bạn đang can thiệp vào có tồn tại hay không là good practice.
- Đoạn sau có nghĩa là, mình lấy ra hàm `onnear` của `inst.components.playerprox`, lưu tạm nó vào 1 biến tên `oldOnNearFn`. Sau đó mình định nghĩa 1 hàm mới `onnearfn`, nhận vào tham số giống hệt hàm `onnear`. Ở đây mình kiểm tra `target` có phải là `wilson` không, nếu có, return ngay lập tức, tức là không làm gì cả. Nếu không phải `wilson`, thực thi hàm `onnear` cũ. Sau đó mình gọi `SetOnPlayerNear` với hàm mới `onnearfn`, thay thế hàm `onnear` cũ.

Đây là 1 kĩ thuật tốt để tránh conflict mod, bạn không trực tiếp set hàm onnear của `inst.components.playerprox` thành 1 hàm mới, mà bạn lấy hàm cũ ra, tạo 1 hàm bọc quanh hàm cũ này, và gán lại hàm mới vào component `playerprox`. Bạn hoàn toàn có thể code như sau:

```
local function WaspHivePostInit(inst)
  if inst.components.playerprox then
    local onnearfn = function(inst, target)
      if target and target.prefab == 'wilson' then
        return
      end

      if inst.components.childspawner ~= nil then
        inst.components.childspawner:ReleaseAllChildren(target, "killerbee")
    	end
    end

    inst.components.playerprox:SetOnPlayerNear(onnearfn)
  end
end
```

Ở đây bạn copy y nguyên hàm `onnear` hiện tại, chèn thêm 1 đoạn if cho `wilson` ở đầu. OK nó sẽ hoạt động khi chỉ có 1 mình mod này, nhưng hãy tưởng tượng có 1 mod khác làm tương tự cho `wendy`, khi đó sẽ chỉ có 1 trong 2 mod hoạt động. Mặt khác, do bạn copy nguyên hàm `onnear` ra hàm mới của bạn, giả sử DST developer cập nhật hàm `onnear`, bạn sẽ không có các cập nhật mới này.

Vậy là xong. Sau đó bạn hãy copy thư mục `mod_template_simple` vào thư mục `Steam/steamapps/common/Don't Starve Together/mods`, và khởi động DST. Tạo 1 world mới, thêm mod này vào, pick Wilson và test theo cách của bạn. Mình thường sẽ sử dụng lệnh `c_gonext("wasphive")` để nhảy đến 1 tổ ong.
