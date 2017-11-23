使用vscode打开C项目时，vscode无法找到头文件路径，提示：`configure includePath for better intellisense results`

**解决：**
编辑`~/.vscode/c_cpp_properties.json`
（`ctrl + shift + p`，键入`C/Cpp : Edit Configurations`，这样就打开了配置文件）， 在`includePath`字段添加头文件的路径即可

> 参考
> [Visual Studio Code includePath](https://stackoverflow.com/questions/37522462/visual-studio-code-includepath)