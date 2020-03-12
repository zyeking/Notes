# 2. 用宏自动创建数据时间

```excel
Private Sub Worksheet_Change(ByVal Target As Range)
'______说明↓___________________________________
'某列单元格发生变化
'若变化值不为空，对应列添加变化时间点日期时间
'若变化值为空，删除对应单元格数据
'示例为A列第2行发生变化，对应的B列添加或删除日期
'______说明↑___________________________________
Application.ScreenUpdating = False '关闭屏幕刷新
Dim cA, cB, startRG As String
Dim offsetc As Long
Dim rg As Range
'______设置参数↓_________________
cA = "A" '变化区域所在列
cB = "B" '日期生成列
startRG = "A1" '变化区域首单元格(防止改动表头触发事件)
'______设置参数↑_________________
offsetc = Columns(cB).Column - Columns(cA).Column
If Not Application.Intersect(Target, Columns(cA), Range(startRG, ActiveCell.SpecialCells(xlLastCell))) Is Nothing Then
    For Each rg In Intersect(Target, Columns(cA), Range(startRG, ActiveCell.SpecialCells(xlLastCell)))
        If rg <> "" Then
            With rg.Offset(0, offsetc)
                .Value = Now
                .NumberFormatLocal = "yyyy/m/d h:mm:ss;@"
            End With
        Else
            rg.Offset(0, offsetc).Clear
        End If
    Next rg
End If
Application.ScreenUpdating = True '恢复屏幕刷新
End Sub
```

