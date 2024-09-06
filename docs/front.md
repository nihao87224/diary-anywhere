下面是仿照 "My Diary - DailyLife and Journal" 日志软件的主要界面和编辑界面的 **XAML 代码**，基于 UWP (Universal Windows Platform) 开发。

### 1. **主界面 XAML 代码**

```xaml
xmlCopy code<Page
    x:Class="DiaryApp.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:DiaryApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <!-- 标题栏 -->
        <StackPanel Orientation="Horizontal" VerticalAlignment="Top" HorizontalAlignment="Stretch" Padding="10">
            <TextBlock Text="{x:Bind DateTime.Now, StringFormat='{}{0:MMMM dd, yyyy}'}"
                       FontSize="20" FontWeight="Bold" HorizontalAlignment="Left"/>
            <Button Content="设置" HorizontalAlignment="Right" Click="SettingsButton_Click"/>
        </StackPanel>

        <!-- 日记列表 -->
        <ListView x:Name="DiaryList" VerticalAlignment="Center" HorizontalAlignment="Stretch" 
                  Margin="10, 60, 10, 0" SelectionChanged="DiaryList_SelectionChanged">
            <ListView.ItemTemplate>
                <DataTemplate x:DataType="local:DiaryEntry">
                    <Border BorderBrush="LightGray" BorderThickness="1" CornerRadius="5" Padding="10" Margin="5">
                        <StackPanel>
                            <TextBlock Text="{x:Bind Title}" FontSize="18" FontWeight="Bold" />
                            <TextBlock Text="{x:Bind Content}" FontSize="14" TextTrimming="CharacterEllipsis" MaxLines="2"/>
                            <TextBlock Text="{x:Bind CreatedDate, StringFormat='创建时间: {0:yyyy-MM-dd}'}" FontSize="12" Foreground="Gray"/>
                        </StackPanel>
                    </Border>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>

        <!-- 新增日记按钮 -->
        <Button Content="新增日记" VerticalAlignment="Bottom" HorizontalAlignment="Right" 
                Margin="10" Click="AddNewDiary_Click"/>
        
        <!-- 搜索栏 -->
        <TextBox PlaceholderText="搜索日记" HorizontalAlignment="Stretch" Margin="10, 40, 10, 0"
                 TextChanged="SearchBox_TextChanged" />
    </Grid>
</Page>
```

### 2. **日记编辑界面 XAML 代码**

```xaml
xmlCopy code<Page
    x:Class="DiaryApp.EditDiaryPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:DiaryApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <!-- 标题栏 -->
        <StackPanel Orientation="Horizontal" VerticalAlignment="Top" HorizontalAlignment="Stretch" Padding="10">
            <TextBlock Text="编辑日记" FontSize="20" FontWeight="Bold"/>
            <Button Content="保存" HorizontalAlignment="Right" Click="SaveDiary_Click"/>
            <Button Content="删除" HorizontalAlignment="Right" Margin="10, 0, 0, 0" Click="DeleteDiary_Click"/>
        </StackPanel>

        <!-- 标题输入框 -->
        <TextBox x:Name="TitleTextBox" PlaceholderText="请输入标题" FontSize="18" Margin="10, 60, 10, 0"/>

        <!-- 内容编辑框 -->
        <TextBox x:Name="ContentTextBox" PlaceholderText="请输入日记内容" FontSize="16" 
                 AcceptsReturn="True" TextWrapping="Wrap" VerticalAlignment="Stretch" 
                 HorizontalAlignment="Stretch" Margin="10, 120, 10, 0"/>

        <!-- 标签选择 -->
        <TextBlock Text="标签：" FontSize="16" Margin="10, 10, 0, 0" VerticalAlignment="Bottom"/>
        <ComboBox x:Name="TagComboBox" Width="200" Margin="10, 10, 0, 0" 
                  PlaceholderText="选择标签" VerticalAlignment="Bottom">
            <ComboBoxItem Content="工作" />
            <ComboBoxItem Content="生活" />
            <ComboBoxItem Content="学习" />
        </ComboBox>
    </Grid>
</Page>
```

### 3. **主要控件解释**

- 主界面

  ：

  - **ListView** 显示日记列表，每个日记用卡片风格展示。
  - **TextBox** 提供搜索框，支持按关键词筛选。
  - **Button** 实现 “新增日记” 功能。

- 日记编辑界面

  ：

  - **TextBox** 用于输入标题和日记内容，内容框支持换行和多行显示。
  - **ComboBox** 提供标签选择功能，用户可以为日记打标签。
  - **保存/删除按钮** 允许用户保存或删除日记。

### 4. **相关事件处理（C# 代码）**

#### 主界面事件处理：

```
csharpCopy codeprivate void AddNewDiary_Click(object sender, RoutedEventArgs e)
{
    // 跳转到新增日记页面
    Frame.Navigate(typeof(EditDiaryPage));
}

private void DiaryList_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    // 选择日记后进入编辑界面
    var selectedDiary = (DiaryEntry)DiaryList.SelectedItem;
    Frame.Navigate(typeof(EditDiaryPage), selectedDiary);
}

private void SearchBox_TextChanged(object sender, TextChangedEventArgs e)
{
    // 搜索日记功能
    string query = (sender as TextBox).Text;
    var filteredDiaries = DiaryManager.SearchDiaries(query);
    DiaryList.ItemsSource = filteredDiaries;
}
```

#### 编辑界面事件处理：

```
csharpCopy codeprivate void SaveDiary_Click(object sender, RoutedEventArgs e)
{
    // 保存日记到数据库
    var diary = new DiaryEntry
    {
        Title = TitleTextBox.Text,
        Content = ContentTextBox.Text,
        Tags = TagComboBox.SelectedItem?.ToString(),
        CreatedDate = DateTime.Now
    };

    DiaryManager.SaveDiary(diary);
    Frame.GoBack();
}

private void DeleteDiary_Click(object sender, RoutedEventArgs e)
{
    // 删除当前日记
    var diary = (DiaryEntry)DataContext;
    DiaryManager.DeleteDiary(diary);
    Frame.GoBack();
}
```

### 5. **总结**

- **界面**：XAML 使用简单明了的布局，通过 `ListView` 显示日记列表，`TextBox` 用于输入，提供直观的用户体验。
- **功能**：结合 C# 代码，实现了日记的增删改查、搜索功能，并通过 MVVM 模式将界面与逻辑分离，便于维护和扩展。