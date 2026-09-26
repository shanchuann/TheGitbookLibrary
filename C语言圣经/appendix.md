---
description: 数据类型、格式字符串、ASCII 和 limits.h。
icon: code
---

# 附件

> **学习路径**：附件不是孤立的资料堆，而是前文查阅工具：类型表帮助检查对象表示，格式字符串表帮助核对 `printf`/`scanf`，ASCII 和 `limits.h` 表帮助验证边界。遇到编译器或平台差异时，优先回到标准和实现说明，而不是背诵某一次运行结果。

### 数据类型

#### 整数类型

| 类型             | 存储大小     | 值范围                                               |
| -------------- | -------- | ------------------------------------------------- |
| char           | 1 字节     | -128 到 127 或 0 到 255                              |
| unsigned char  | 1 字节     | 0 到 255                                           |
| signed char    | 1 字节     | -128 到 127                                        |
| int            | 2 或 4 字节 | -32,768 到 32,767 或 -2,147,483,648 到 2,147,483,647 |
| unsigned int   | 2 或 4 字节 | 0 到 65,535 或 0 到 4,294,967,295                    |
| short          | 2 字节     | -32,768 到 32,767                                  |
| unsigned short | 2 字节     | 0 到 65,535                                        |
| long           | 4 字节     | -2,147,483,648 到 2,147,483,647                    |
| unsigned long  | 4 字节     | 0 到 4,294,967,295                                 |

#### 浮点类型

| 类型          | 存储大小  | 值范围                   | 精度      |
| ----------- | ----- | --------------------- | ------- |
| float       | 4 字节  | 1.2E-38 到 3.4E+38     | 6 位有效位  |
| double      | 8 字节  | 2.3E-308 到 1.7E+308   | 15 位有效位 |
| long double | 16 字节 | 3.4E-4932 到 1.1E+4932 | 19 位有效位 |

#### void 类型

<table><thead><tr><th width="118.199951171875">序号</th><th>类型与描述</th></tr></thead><tbody><tr><td>1</td><td><strong>函数返回为空</strong> C 中有各种函数都不返回值，或者您可以说它们返回空。不返回值的函数的返回类型为空。例如 <strong>void exit (int status);</strong></td></tr><tr><td>2</td><td><strong>函数参数为空</strong> C 中有各种函数不接受任何参数。不带参数的函数可以接受一个 void。例如 <strong>int rand(void);</strong></td></tr><tr><td>3</td><td><strong>指针指向 void</strong> 类型为 void * 的指针代表对象的地址，而不是类型。例如，内存分配函数 <strong>void *malloc( size_t size );</strong> 返回指向 void 的指针，可以转换为任何数据类型。</td></tr></tbody></table>

### 常用格式字符串

#### 整数类

| 格式说明符           | 对应数据类型                 | 说明/示例                              |
| --------------- | ---------------------- | ---------------------------------- |
| `%d` / `%i`     | `int`                  | 有符号十进制整数（`%i`在`scanf`中可识别八/十六进制）   |
| `%u`            | `unsigned int`         | 无符号十进制整数                           |
| `%o`            | `unsigned int`         | 无符号八进制整数（无前缀`0`）                   |
| `%x` / `%X`     | `unsigned int`         | 无符号十六进制整数（`%x`小写`a-f`，`%X`大写`A-F`） |
| `%hd`           | `short`                | 短整型有符号十进制                          |
| `%hu`           | `unsigned short`       | 短整型无符号十进制                          |
| `%ld` / `%li`   | `long`                 | 长整型有符号十进制                          |
| `%lu`           | `unsigned long`        | 长整型无符号十进制                          |
| `%lld` / `%lli` | `long long`            | 长长整型有符号十进制 (C99)                   |
| `%llu`          | `unsigned long long`   | 长长整型无符号十进制 (C99)                   |
| `%zd` / `%zi`   | `size_t` / `ptrdiff_t` | 指针大小类型/指针差值类型 (C99)                |

#### 浮点类

| 格式说明符       | 对应数据类型             | 说明/示例                                  |
| ----------- | ------------------ | -------------------------------------- |
| `%f`        | `float` / `double` | 十进制小数形式（`printf`中`float`自动提升为`double`） |
| `%lf`       | `double`           | 仅用于 **`scanf`** 读取 `double` 类型         |
| `%e` / `%E` | `float` / `double` | 科学计数法（`%e`小写`e`，`%E`大写`E`）             |
| `%g` / `%G` | `float` / `double` | 自动选`%f`或`%e`中更短形式（去掉尾部无效0）             |
| `%a` / `%A` | `float` / `double` | 十六进制科学计数法 (C99)                        |
| `%Lf`       | `long double`      | 长双精度浮点数                                |

#### 字符与字符串

| 格式说明符  | 对应数据类型                  | 说明/示例                                    |
| ------ | ----------------------- | ---------------------------------------- |
| `%c`   | `char` / `int`          | 单个字符（`printf`中`int`会转成`unsigned char`输出） |
| `%s`   | `char*` / `const char*` | 字符串（必须以 `'\0'` 结尾）                       |
| `%hhd` | `signed char`           | 有符号字符型整数 (C99)                           |
| `%hhu` | `unsigned char`         | 无符号字符型整数 (C99)                           |

#### 指针与特殊

| 格式说明符 | 对应数据类型  | 说明/示例                       |
| ----- | ------- | --------------------------- |
| `%p`  | `void*` | 指针地址（通常以十六进制显示，建议强转`void*`） |
| `%n`  | `int*`  | 不输出内容，将**已输出的字符数**写入对应指针变量  |
| `%%`  | 无       | 输出一个字面百分号 `%`               |

#### 常用修饰符

| 修饰符类型  | 示例     | 说明                             |
| ------ | ------ | ------------------------------ |
| **标志** | `%-5d` | `-`：左对齐（默认右对齐）                 |
| **标志** | `%+d`  | `+`：强制显示正负号（正数也显示 `+`）         |
| **标志** | `% d`  | 空格：正数前补空格，负数前显示 `-`            |
| **标志** | `%#x`  | `#`：八进制加前缀 `0`，十六进制加 `0x`/`0X` |
| **宽度** | `%5d`  | 最小宽度 5，不足则补空格                  |
| **精度** | `%.2f` | 浮点数保留 2 位小数；字符串最多输出 2 个字符      |
| **长度** | `%lld` | `ll`：修饰整型，对应 `long long`       |

### ASCII表

<table><thead><tr><th width="105.5999755859375">二进制</th><th width="70">八进制</th><th width="70.599853515625">十进制</th><th width="78.800048828125">十六进制</th><th width="242.800048828125">字符/缩写</th><th>解释</th></tr></thead><tbody><tr><td>00000000</td><td>000</td><td>0</td><td>00</td><td>NUL (NULL)</td><td>空字符</td></tr><tr><td>00000001</td><td>001</td><td>1</td><td>01</td><td>SOH (Start Of Headling)</td><td>标题开始</td></tr><tr><td>00000010</td><td>002</td><td>2</td><td>02</td><td>STX (Start Of Text)</td><td>正文开始</td></tr><tr><td>00000011</td><td>003</td><td>3</td><td>03</td><td>ETX (End Of Text)</td><td>正文结束</td></tr><tr><td>00000100</td><td>004</td><td>4</td><td>04</td><td>EOT (End Of Transmission)</td><td>传输结束</td></tr><tr><td>00000101</td><td>005</td><td>5</td><td>05</td><td>ENQ (Enquiry)</td><td>请求</td></tr><tr><td>00000110</td><td>006</td><td>6</td><td>06</td><td>ACK (Acknowledge)</td><td>回应/响应/收到通知</td></tr><tr><td>00000111</td><td>007</td><td>7</td><td>07</td><td>BEL (Bell)</td><td>响铃</td></tr><tr><td>00001000</td><td>010</td><td>8</td><td>08</td><td>BS (Backspace)</td><td>退格</td></tr><tr><td>00001001</td><td>011</td><td>9</td><td>09</td><td>HT (Horizontal Tab)</td><td>水平制表符</td></tr><tr><td>00001010</td><td>012</td><td>10</td><td>0A</td><td>LF/NL(Line Feed/New Line)</td><td>换行键</td></tr><tr><td>00001011</td><td>013</td><td>11</td><td>0B</td><td>VT (Vertical Tab)</td><td>垂直制表符</td></tr><tr><td>00001100</td><td>014</td><td>12</td><td>0C</td><td>FF/NP (Form Feed/New Page)</td><td>换页键</td></tr><tr><td>00001101</td><td>015</td><td>13</td><td>0D</td><td>CR (Carriage Return)</td><td>回车键</td></tr><tr><td>00001110</td><td>016</td><td>14</td><td>0E</td><td>SO (Shift Out)</td><td>不用切换</td></tr><tr><td>00001111</td><td>017</td><td>15</td><td>0F</td><td>SI (Shift In)</td><td>启用切换</td></tr><tr><td>00010000</td><td>020</td><td>16</td><td>10</td><td>DLE (Data Link Escape)</td><td>数据链路转义</td></tr><tr><td>00010001</td><td>021</td><td>17</td><td>11</td><td>DC1/XON (Device Control 1/Transmission On)</td><td>设备控制1/传输开始</td></tr><tr><td>00010010</td><td>022</td><td>18</td><td>12</td><td>DC2 (Device Control 2)</td><td>设备控制2</td></tr><tr><td>00010011</td><td>023</td><td>19</td><td>13</td><td>DC3/XOFF (Device Control 3/Transmission Off)</td><td>设备控制3/传输中断</td></tr><tr><td>00010100</td><td>024</td><td>20</td><td>14</td><td>DC4 (Device Control 4)</td><td>设备控制4</td></tr><tr><td>00010101</td><td>025</td><td>21</td><td>15</td><td>NAK (Negative Acknowledge)</td><td>无响应/非正常响应/拒绝接收</td></tr><tr><td>00010110</td><td>026</td><td>22</td><td>16</td><td>SYN (Synchronous Idle)</td><td>同步空闲</td></tr><tr><td>00010111</td><td>027</td><td>23</td><td>17</td><td>ETB (End of Transmission Block)</td><td>传输块结束/块传输终止</td></tr><tr><td>00011000</td><td>030</td><td>24</td><td>18</td><td>CAN (Cancel)</td><td>取消</td></tr><tr><td>00011001</td><td>031</td><td>25</td><td>19</td><td>EM (End of Medium)</td><td>已到介质末端/介质存储已满/介质中断</td></tr><tr><td>00011010</td><td>032</td><td>26</td><td>1A</td><td>SUB (Substitute)</td><td>替补/替换</td></tr><tr><td>00011011</td><td>033</td><td>27</td><td>1B</td><td>ESC (Escape)</td><td>逃离/取消</td></tr><tr><td>00011100</td><td>034</td><td>28</td><td>1C</td><td>FS (File Separator)</td><td>文件分割符</td></tr><tr><td>00011101</td><td>035</td><td>29</td><td>1D</td><td>GS (Group Separator)</td><td>组分隔符/分组符</td></tr><tr><td>00011110</td><td>036</td><td>30</td><td>1E</td><td>RS (Record Separator)</td><td>记录分离符</td></tr><tr><td>00011111</td><td>037</td><td>31</td><td>1F</td><td>US (Unit Separator)</td><td>单元分隔符</td></tr><tr><td>00100000</td><td>040</td><td>32</td><td>20</td><td>(Space)</td><td>空格</td></tr><tr><td>00100001</td><td>041</td><td>33</td><td>21</td><td>!</td><td></td></tr><tr><td>00100010</td><td>042</td><td>34</td><td>22</td><td>"</td><td></td></tr><tr><td>00100011</td><td>043</td><td>35</td><td>23</td><td>#</td><td></td></tr><tr><td>00100100</td><td>044</td><td>36</td><td>24</td><td>$</td><td></td></tr><tr><td>00100101</td><td>045</td><td>37</td><td>25</td><td>%</td><td></td></tr><tr><td>00100110</td><td>046</td><td>38</td><td>26</td><td>&#x26;</td><td></td></tr><tr><td>00100111</td><td>047</td><td>39</td><td>27</td><td>'</td><td></td></tr><tr><td>00101000</td><td>050</td><td>40</td><td>28</td><td>(</td><td></td></tr><tr><td>00101001</td><td>051</td><td>41</td><td>29</td><td>)</td><td></td></tr><tr><td>00101010</td><td>052</td><td>42</td><td>2A</td><td>*</td><td></td></tr><tr><td>00101011</td><td>053</td><td>43</td><td>2B</td><td>+</td><td></td></tr><tr><td>00101100</td><td>054</td><td>44</td><td>2C</td><td>,</td><td></td></tr><tr><td>00101101</td><td>055</td><td>45</td><td>2D</td><td>-</td><td></td></tr><tr><td>00101110</td><td>056</td><td>46</td><td>2E</td><td>.</td><td></td></tr><tr><td>00101111</td><td>057</td><td>47</td><td>2F</td><td>/</td><td></td></tr><tr><td>00110000</td><td>060</td><td>48</td><td>30</td><td>0</td><td></td></tr><tr><td>00110001</td><td>061</td><td>49</td><td>31</td><td>1</td><td></td></tr><tr><td>00110010</td><td>062</td><td>50</td><td>32</td><td>2</td><td></td></tr><tr><td>00110011</td><td>063</td><td>51</td><td>33</td><td>3</td><td></td></tr><tr><td>00110100</td><td>064</td><td>52</td><td>34</td><td>4</td><td></td></tr><tr><td>00110101</td><td>065</td><td>53</td><td>35</td><td>5</td><td></td></tr><tr><td>00110110</td><td>066</td><td>54</td><td>36</td><td>6</td><td></td></tr><tr><td>00110111</td><td>067</td><td>55</td><td>37</td><td>7</td><td></td></tr><tr><td>00111000</td><td>070</td><td>56</td><td>38</td><td>8</td><td></td></tr><tr><td>00111001</td><td>071</td><td>57</td><td>39</td><td>9</td><td></td></tr><tr><td>00111010</td><td>072</td><td>58</td><td>3A</td><td>:</td><td></td></tr><tr><td>00111011</td><td>073</td><td>59</td><td>3B</td><td>;</td><td></td></tr><tr><td>00111100</td><td>074</td><td>60</td><td>3C</td><td>&#x3C;</td><td></td></tr><tr><td>00111101</td><td>075</td><td>61</td><td>3D</td><td>=</td><td></td></tr><tr><td>00111110</td><td>076</td><td>62</td><td>3E</td><td>></td><td></td></tr><tr><td>00111111</td><td>077</td><td>63</td><td>3F</td><td>?</td><td></td></tr><tr><td>01000000</td><td>100</td><td>64</td><td>40</td><td>@</td><td></td></tr><tr><td>01000001</td><td>101</td><td>65</td><td>41</td><td>A</td><td></td></tr><tr><td>01000010</td><td>102</td><td>66</td><td>42</td><td>B</td><td></td></tr><tr><td>01000011</td><td>103</td><td>67</td><td>43</td><td>C</td><td></td></tr><tr><td>01000100</td><td>104</td><td>68</td><td>44</td><td>D</td><td></td></tr><tr><td>01000101</td><td>105</td><td>69</td><td>45</td><td>E</td><td></td></tr><tr><td>01000110</td><td>106</td><td>70</td><td>46</td><td>F</td><td></td></tr><tr><td>01000111</td><td>107</td><td>71</td><td>47</td><td>G</td><td></td></tr><tr><td>01001000</td><td>110</td><td>72</td><td>48</td><td>H</td><td></td></tr><tr><td>01001001</td><td>111</td><td>73</td><td>49</td><td>I</td><td></td></tr><tr><td>01001010</td><td>112</td><td>74</td><td>4A</td><td>J</td><td></td></tr><tr><td>01001011</td><td>113</td><td>75</td><td>4B</td><td>K</td><td></td></tr><tr><td>01001100</td><td>114</td><td>76</td><td>4C</td><td>L</td><td></td></tr><tr><td>01001101</td><td>115</td><td>77</td><td>4D</td><td>M</td><td></td></tr><tr><td>01001110</td><td>116</td><td>78</td><td>4E</td><td>N</td><td></td></tr><tr><td>01001111</td><td>117</td><td>79</td><td>4F</td><td>O</td><td></td></tr><tr><td>01010000</td><td>120</td><td>80</td><td>50</td><td>P</td><td></td></tr><tr><td>01010001</td><td>121</td><td>81</td><td>51</td><td>Q</td><td></td></tr><tr><td>01010010</td><td>122</td><td>82</td><td>52</td><td>R</td><td></td></tr><tr><td>01010011</td><td>123</td><td>83</td><td>53</td><td>S</td><td></td></tr><tr><td>01010100</td><td>124</td><td>84</td><td>54</td><td>T</td><td></td></tr><tr><td>01010101</td><td>125</td><td>85</td><td>55</td><td>U</td><td></td></tr><tr><td>01010110</td><td>126</td><td>86</td><td>56</td><td>V</td><td></td></tr><tr><td>01010111</td><td>127</td><td>87</td><td>57</td><td>W</td><td></td></tr><tr><td>01011000</td><td>130</td><td>88</td><td>58</td><td>X</td><td></td></tr><tr><td>01011001</td><td>131</td><td>89</td><td>59</td><td>Y</td><td></td></tr><tr><td>01011010</td><td>132</td><td>90</td><td>5A</td><td>Z</td><td></td></tr><tr><td>01011011</td><td>133</td><td>91</td><td>5B</td><td>[</td><td></td></tr><tr><td>01011100</td><td>134</td><td>92</td><td>5C</td><td>\</td><td></td></tr><tr><td>01011101</td><td>135</td><td>93</td><td>5D</td><td>]</td><td></td></tr><tr><td>01011110</td><td>136</td><td>94</td><td>5E</td><td>^</td><td></td></tr><tr><td>01011111</td><td>137</td><td>95</td><td>5F</td><td>_</td><td></td></tr><tr><td>01100000</td><td>140</td><td>96</td><td>60</td><td>`</td><td></td></tr><tr><td>01100001</td><td>141</td><td>97</td><td>61</td><td>a</td><td></td></tr><tr><td>01100010</td><td>142</td><td>98</td><td>62</td><td>b</td><td></td></tr><tr><td>01100011</td><td>143</td><td>99</td><td>63</td><td>c</td><td></td></tr><tr><td>01100100</td><td>144</td><td>100</td><td>64</td><td>d</td><td></td></tr><tr><td>01100101</td><td>145</td><td>101</td><td>65</td><td>e</td><td></td></tr><tr><td>01100110</td><td>146</td><td>102</td><td>66</td><td>f</td><td></td></tr><tr><td>01100111</td><td>147</td><td>103</td><td>67</td><td>g</td><td></td></tr><tr><td>01101000</td><td>150</td><td>104</td><td>68</td><td>h</td><td></td></tr><tr><td>01101001</td><td>151</td><td>105</td><td>69</td><td>i</td><td></td></tr><tr><td>01101010</td><td>152</td><td>106</td><td>6A</td><td>j</td><td></td></tr><tr><td>01101011</td><td>153</td><td>107</td><td>6B</td><td>k</td><td></td></tr><tr><td>01101100</td><td>154</td><td>108</td><td>6C</td><td>l</td><td></td></tr><tr><td>01101101</td><td>155</td><td>109</td><td>6D</td><td>m</td><td></td></tr><tr><td>01101110</td><td>156</td><td>110</td><td>6E</td><td>n</td><td></td></tr><tr><td>01101111</td><td>157</td><td>111</td><td>6F</td><td>o</td><td></td></tr><tr><td>01110000</td><td>160</td><td>112</td><td>70</td><td>p</td><td></td></tr><tr><td>01110001</td><td>161</td><td>113</td><td>71</td><td>q</td><td></td></tr><tr><td>01110010</td><td>162</td><td>114</td><td>72</td><td>r</td><td></td></tr><tr><td>01110011</td><td>163</td><td>115</td><td>73</td><td>s</td><td></td></tr><tr><td>01110100</td><td>164</td><td>116</td><td>74</td><td>t</td><td></td></tr><tr><td>01110101</td><td>165</td><td>117</td><td>75</td><td>u</td><td></td></tr><tr><td>01110110</td><td>166</td><td>118</td><td>76</td><td>v</td><td></td></tr><tr><td>01110111</td><td>167</td><td>119</td><td>77</td><td>w</td><td></td></tr><tr><td>01111000</td><td>170</td><td>120</td><td>78</td><td>x</td><td></td></tr><tr><td>01111001</td><td>171</td><td>121</td><td>79</td><td>y</td><td></td></tr><tr><td>01111010</td><td>172</td><td>122</td><td>7A</td><td>z</td><td></td></tr><tr><td>01111011</td><td>173</td><td>123</td><td>7B</td><td>{</td><td></td></tr><tr><td>01111100</td><td>174</td><td>124</td><td>7C</td><td>|</td><td></td></tr><tr><td>01111101</td><td>175</td><td>125</td><td>7D</td><td>}</td><td></td></tr><tr><td>01111110</td><td>176</td><td>126</td><td>7E</td><td>~</td><td></td></tr><tr><td>01111111</td><td>177</td><td>127</td><td>7F</td><td>DEL (Delete)</td><td>删除</td></tr></tbody></table>

### C 标准库 `<limits.h>`

| **宏**        | **描述**                      | **值**                   |
| ------------ | --------------------------- | ----------------------- |
| **字符类型**     |                             |                         |
| `CHAR_BIT`   | `char` 类型的位数                | 通常为 8                   |
| `CHAR_MIN`   | `char` 类型的最小值（有符号或无符号）      | -128 或 0                |
| `CHAR_MAX`   | `char` 类型的最大值（有符号或无符号）      | 127 或 255               |
| `SCHAR_MIN`  | `signed char` 类型的最小值        | -128                    |
| `SCHAR_MAX`  | `signed char` 类型的最大值        | 127                     |
| `UCHAR_MAX`  | `unsigned char` 类型的最大值      | 255                     |
| **短整数类型**    |                             |                         |
| `SHRT_MIN`   | `short` 类型的最小值              | -32768                  |
| `SHRT_MAX`   | `short` 类型的最大值              | 32767                   |
| `USHRT_MAX`  | `unsigned short` 类型的最大值     | 65535                   |
| **整数类型**     |                             |                         |
| `INT_MIN`    | `int` 类型的最小值                | -2147483648             |
| `INT_MAX`    | `int` 类型的最大值                | 2147483647              |
| `UINT_MAX`   | `unsigned int` 类型的最大值       | 4294967295              |
| **长整数类型**    |                             |                         |
| `LONG_MIN`   | `long` 类型的最小值               | -9223372036854775808L   |
| `LONG_MAX`   | `long` 类型的最大值               | 9223372036854775807L    |
| `ULONG_MAX`  | `unsigned long` 类型的最大值      | 18446744073709551615UL  |
| **长长整数类型**   |                             |                         |
| `LLONG_MIN`  | `long long` 类型的最小值          | -9223372036854775808LL  |
| `LLONG_MAX`  | `long long` 类型的最大值          | 9223372036854775807LL   |
| `ULLONG_MAX` | `unsigned long long` 类型的最大值 | 18446744073709551615ULL |

## 编译器速查

示例默认使用 C11：

```sh
gcc -std=c11 -Wall -Wextra -Wpedantic -g source.c -o program
```

需要固定宽度整数时使用 `<stdint.h>`，打印 `uint32_t` 等类型时使用 `<inttypes.h>` 提供的格式宏。不要把某个平台上 `int`、`long` 或指针的大小写进可移植文件格式。
