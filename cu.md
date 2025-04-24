说明：wpf路径迷宫鼠标绘制
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9fa79c982aa5439b8fc9a2d45cbeb3d3.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MainWindow.xaml.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Shapes;

namespace WpfApp1
{
    public partial class MainWindow : Window
    {
        private const int CellSize = 30;
        private List<Point> _path = new();
        private List<Point> _userPath = new();
        private Cell[,] _maze = new Cell[0, 0];
        private bool _isDrawing;

        public MainWindow()
        {
            InitializeComponent();
        }

        #region Maze Drawing
        private void DrawMaze()
        {
            MazeCanvas.Children.Clear();
            if (_maze == null) return;

            // Draw walls
            for (int y = 0; y < _maze.GetLength(0); y++)
                for (int x = 0; x < _maze.GetLength(1); x++)
                    DrawWalls(x, y, _maze[y, x]);

            // Draw start/end markers
            DrawCircle(0, 0, Brushes.Green);
            DrawCircle(_maze.GetLength(1) - 1, _maze.GetLength(0) - 1, Brushes.Red);
        }

        private void DrawWalls(int x, int y, Cell cell)
        {
            var x1 = x * CellSize;
            var y1 = y * CellSize;
            
            if (cell.North) DrawLine(x1, y1, x1 + CellSize, y1);
            if (cell.South) DrawLine(x1, y1 + CellSize, x1 + CellSize, y1 + CellSize);
            if (cell.West) DrawLine(x1, y1, x1, y1 + CellSize);
            if (cell.East) DrawLine(x1 + CellSize, y1, x1 + CellSize, y1 + CellSize);
        }

        private void DrawLine(double x1, double y1, double x2, double y2)
        {
            MazeCanvas.Children.Add(new Line
            {
                X1 = x1,
                Y1 = y1,
                X2 = x2,
                Y2 = y2,
                Stroke = Brushes.Black,
                StrokeThickness = 1
            });
        }
        #endregion

        #region Path Handling
        private void DrawUserPath()
        {
            // Clear existing green lines
            var oldLines = MazeCanvas.Children.OfType<Line>()
                .Where(l => l.Stroke == Brushes.Green).ToList();
            oldLines.ForEach(l => MazeCanvas.Children.Remove(l));

            // Draw new path
            for (int i = 1; i < _userPath.Count; i++)
            {
                var prev = _userPath[i - 1];
                var curr = _userPath[i];
                
                MazeCanvas.Children.Add(new Line
                {
                    X1 = (prev.X + 0.5) * CellSize,
                    Y1 = (prev.Y + 0.5) * CellSize,
                    X2 = (curr.X + 0.5) * CellSize,
                    Y2 = (curr.Y + 0.5) * CellSize,
                    Stroke = Brushes.Green,
                    StrokeThickness = 3
                });
            }
        }

        private bool ValidatePath()
        {
            // Check if reached end
            var end = new Point(_maze.GetLength(1)-1, _maze.GetLength(0)-1);
            if (_userPath.LastOrDefault() != end) return false;

            // Validate each step
            for (int i = 1; i < _userPath.Count; i++)
            {
                var prev = _userPath[i - 1];
                var curr = _userPath[i];
                
                if (!IsValidMove((int)prev.X, (int)prev.Y, 
                               (int)curr.X, (int)curr.Y))
                    return false;
            }
            return true;
        }

        private bool IsValidMove(int fromX, int fromY, int toX, int toY)
        {
            var cell = _maze[fromY, fromX];
            var dx = toX - fromX;
            var dy = toY - fromY;

            return (dx, dy) switch
            {
                (1, 0) => !cell.East,   // Right
                (-1, 0) => !cell.West,  // Left
                (0, 1) => !cell.South,  // Down
                (0, -1) => !cell.North, // Up
                _ => false
            };
        }
        #endregion

        #region Mouse Handling
        private void MazeCanvas_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            if (_maze == null) return;
            
            var pos = e.GetPosition(MazeCanvas);
            int x = (int)(pos.X / CellSize);
            int y = (int)(pos.Y / CellSize);

            // Must start from origin
            if (x == 0 && y == 0)
            {
                _isDrawing = true;
                _userPath.Clear();
                _userPath.Add(new Point(x, y));
                DrawUserPath();
            }
        }

        private void MazeCanvas_MouseMove(object sender, MouseEventArgs e)
        {
            if (!_isDrawing || _maze == null) return;

            var pos = e.GetPosition(MazeCanvas);
            int x = (int)(pos.X / CellSize);
            int y = (int)(pos.Y / CellSize);

            // Validate cell position
            if (x < 0 || x >= _maze.GetLength(1) || 
                y < 0 || y >= _maze.GetLength(0)) return;

            var last = _userPath.LastOrDefault();
            
            // Allow only adjacent moves
            if (Math.Abs(x - last.X) + Math.Abs(y - last.Y) != 1) return;
            
            // Prevent backtracking
            if (_userPath.Any(p => p.X == x && p.Y == y)) return;

            if (IsValidMove((int)last.X, (int)last.Y, x, y))
            {
                _userPath.Add(new Point(x, y));
                DrawUserPath();
            }
        }

        private void MazeCanvas_MouseLeftButtonUp(object sender, MouseButtonEventArgs e)
        {
            if (!_isDrawing) return;
            _isDrawing = false;

            bool success = ValidatePath();
            string message = success ? 
                "Congratulations! Restart?" : "Wrong path! Try again?";
            
            if (MessageBox.Show(message, "Result", 
                MessageBoxButton.YesNo, MessageBoxImage.Question) == MessageBoxResult.Yes)
            {
                BtnGenerate_Click(null, null);
            }
            else
            {
                _userPath.Clear();
                DrawUserPath();
            }
        }
        #endregion

        #region UI Events
        private void BtnGenerate_Click(object sender, RoutedEventArgs e)
        {
            // Generate new maze
            _maze = new MazeGenerator().Generate(15, 15);
            
            // Reset state
            _path.Clear();
            _userPath.Clear();
            _isDrawing = false;
            
            // Clear drawings
            MazeCanvas.Children.Clear();
            DrawMaze();
        }

        private void BtnSolve_Click(object sender, RoutedEventArgs e)
        {
            if (_maze == null) return;
            
            // Show auto-solved path
            _path = new PathSolver().SolveBFS(_maze);
            DrawMaze();
            
            // Draw solution path
            foreach (var point in _path)
            {
                MazeCanvas.Children.Add(new Ellipse
                {
                    Width = 4,
                    Height = 4,
                    Fill = Brushes.Blue,
                    Margin = new Thickness(
                        (point.X + 0.5) * CellSize - 2,
                        (point.Y + 0.5) * CellSize - 2, 0, 0)
                });
            }
        }
        #endregion

        #region Helpers
        private void DrawCircle(int x, int y, Brush color)
        {
            MazeCanvas.Children.Add(new Ellipse
            {
                Width = CellSize / 2,
                Height = CellSize / 2,
                Fill = color,
                Margin = new Thickness(
                    x * CellSize + CellSize / 4,
                    y * CellSize + CellSize / 4, 0, 0)
            });
        }
        #endregion
    }
}
```

step2:C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MazeGenerator.cs

```csharp
using System;
using System.Collections.Generic;
using System.Windows;

namespace WpfApp1;

public struct Cell
{
    public bool North;
    public bool South;
    public bool East;
    public bool West;
    public bool Visited;
}

public class MazeGenerator
{
    private Cell[,] _maze = new Cell[0,0]; // 初始化
    private readonly Random _random = new();
    // 修改方向处理
    private readonly (int dx, int dy)[] _directions = 
    {
        (0, -1),  // North
        (0, 1),   // South
        (1, 0),   // East
        (-1, 0)   // West
    };

    public Cell[,] Generate(int width, int height)
    {
        _maze = new Cell[height, width];
        InitializeCells();
        GenerateWithBacktracking();
        return _maze;
    }

    private void InitializeCells()
    {
        for (int y = 0; y < _maze.GetLength(0); y++)
        for (int x = 0; x < _maze.GetLength(1); x++)
            _maze[y, x] = new Cell
            {
                North = true, South = true, 
                East = true, West = true,
                Visited = false
            };
    }

    private void GenerateWithBacktracking()
    {
        var stack = new Stack<(int x, int y)>();
        stack.Push((0, 0));
        _maze[0, 0].Visited = true;

        var directions = new[]
        {
            new Point(0, -1),  // North
            new Point(0, 1),   // South
            new Point(1, 0),   // East
            new Point(-1, 0)   // West
        };

        while (stack.Count > 0)
        {
            var (x, y) = stack.Pop();
            
            var validDirs = _directions
                .Select((d, i) => new { Direction = d, Index = i })
                .Where(d => 
                {
                    var (dx, dy) = d.Direction;
                    int nx = x + dx;
                    int ny = y + dy;
                    return IsValidCell(nx, ny) && !_maze[ny, nx].Visited;
                })
                .ToList();

            if (validDirs.Count > 0)
            {
                var selected = validDirs[_random.Next(validDirs.Count)];
                var (dx, dy) = selected.Direction;
                int nx = x + dx;
                int ny = y + dy;

                RemoveWalls(x, y, selected.Index);
                _maze[ny, nx].Visited = true;
                stack.Push((x, y));
                stack.Push((nx, ny));
            }
        }
    }

    private void RemoveWalls(int x, int y, int dir)
    {
        switch (dir)
        {
            case 0: // North
                _maze[y, x].North = false;
                _maze[y - 1, x].South = false;
                break;
            case 1: // South
                _maze[y, x].South = false;
                _maze[y + 1, x].North = false;
                break;
            case 2: // East
                _maze[y, x].East = false;
                _maze[y, x + 1].West = false;
                break;
            case 3: // West
                _maze[y, x].West = false;
                _maze[y, x - 1].East = false;
                break;
        }
    }

    private bool IsValidCell(int x, int y) => 
        x >= 0 && x < _maze.GetLength(1) && 
        y >= 0 && y < _maze.GetLength(0);
}
```

step3:C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\PathSolver.cs

```csharp
using System;
using System.Collections.Generic;
using System.Windows;

namespace WpfApp1;

public class PathSolver
{
     
    
    public List<Point> SolveBFS(Cell[,] maze)
    {
        int width = maze.GetLength(1);
        int height = maze.GetLength(0);
        var visited = new bool[height, width];
        var prev = new (int x, int y)[height, width];
        var queue = new Queue<(int x, int y)>();
        var directions = new[] { (0, -1), (0, 1), (1, 0), (-1, 0) };

        queue.Enqueue((0, 0));
        visited[0, 0] = true;

        while (queue.Count > 0)
        {
            var (x, y) = queue.Dequeue();
            if (x == width - 1 && y == height - 1)
                return ReconstructPath(prev);

            foreach (var (dx, dy) in directions)
            {
                int nx = x + dx;
                int ny = y + dy;

                if (nx < 0 || nx >= width || ny < 0 || ny >= height) continue;
                if (visited[ny, nx]) continue;
                
                if (!CanMove(maze[y, x], GetDirectionIndex(dx, dy))) continue;

                visited[ny, nx] = true;
                prev[ny, nx] = (x, y);
                queue.Enqueue((nx, ny));
            }
        }
        return new List<Point>();
    }

    
    

    private int GetDirectionIndex(int dx, int dy) => (dx, dy) switch
    {
        (0, -1) => 0,  // North
        (0, 1) => 1,   // South
        (1, 0) => 2,   // East
        (-1, 0) => 3,  // West
        _ => -1
    };



    

    private List<Point> ReconstructPath((int x, int y)[,] prev)
    {
        var path = new List<Point>();
        (int x, int y) current = (prev.GetLength(1) - 1, prev.GetLength(0) - 1);
        
        while (current.x != 0 || current.y != 0)
        {
            path.Add(new Point(current.x, current.y));
            current = prev[current.y, current.x];
        }
        path.Add(new Point(0, 0));
        path.Reverse();
        return path;
    }



    private bool CanMove(Cell cell, int dir) => dir switch
    {
        0 => !cell.North,
        1 => !cell.South,
        2 => !cell.East,
        3 => !cell.West,
        _ => false
    };

    private List<Point> ReconstructPath(Point[,] prev)
    {
        var path = new List<Point>();
        var current = new Point(prev.GetLength(1) - 1, prev.GetLength(0) - 1);
        
        while (current.X != 0 || current.Y != 0)
        {
            path.Add(current);
            current = prev[(int)current.Y, (int)current.X];
        }
        path.Add(new Point(0, 0));
        path.Reverse();
        return path;
    }
}
```

step4:C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MainWindow.xaml

```xml
<Window x:Class="WpfApp1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Maze Generator" Height="600" Width="800">
    <DockPanel>
        <StackPanel DockPanel.Dock="Top" Orientation="Horizontal" Margin="5">
            <Button x:Name="BtnGenerate" Content="Generate Maze" 
                    Click="BtnGenerate_Click" Margin="5" Padding="10 3"/>
            <Button x:Name="BtnSolve" Content="Solve Maze" 
                    Click="BtnSolve_Click" Margin="5" Padding="10 3"/>
        </StackPanel>
        <Canvas x:Name="MazeCanvas" Background="White" Margin="10" ClipToBounds="True"
                MouseLeftButtonDown="MazeCanvas_MouseLeftButtonDown"
                MouseMove="MazeCanvas_MouseMove"
                MouseLeftButtonUp="MazeCanvas_MouseLeftButtonUp"/>
    </DockPanel>
</Window>
```

step5:纯界面

```typescript
   C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MainWindow.xaml.cs
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Shapes;

namespace WpfApp1;

public partial class MainWindow : Window
{
    private const int CellSize = 20;
    private List<Point> _path = new();
    private Cell[,] _maze = new Cell[0,0]; // 初始化为空数组
    
    public MainWindow()
    {
        InitializeComponent();
    }

    private void DrawMaze()
    {
        MazeCanvas.Children.Clear();
        if (_maze == null) return;

        for (int y = 0; y < _maze.GetLength(0); y++)
        {
            for (int x = 0; x < _maze.GetLength(1); x++)
            {
                var cell = _maze[y, x];
                DrawWalls(x, y, cell);
            }
        }
        DrawStartEnd();
    }

    private void DrawWalls(int x, int y, Cell cell)
    {
        var x1 = x * CellSize;
        var y1 = y * CellSize;
        var x2 = (x + 1) * CellSize;
        var y2 = (y + 1) * CellSize;

        if (cell.North) DrawLine(x1, y1, x2, y1);
        if (cell.South) DrawLine(x1, y2, x2, y2);
        if (cell.West) DrawLine(x1, y1, x1, y2);
        if (cell.East) DrawLine(x2, y1, x2, y2);
    }

    private void DrawLine(double x1, double y1, double x2, double y2)
    {
        var line = new Line
        {
            X1 = x1,
            Y1 = y1,
            X2 = x2,
            Y2 = y2,
            Stroke = Brushes.Black,
            StrokeThickness = 1
        };
        MazeCanvas.Children.Add(line);
    }

    private void DrawStartEnd()
    {
        DrawCircle(0, 0, Brushes.Green);
        var lastX = _maze.GetLength(1) - 1;
        var lastY = _maze.GetLength(0) - 1;
        DrawCircle(lastX, lastY, Brushes.Red);
    }

    private void DrawCircle(int x, int y, Brush color)
    {
        var ellipse = new Ellipse
        {
            Width = CellSize / 2,
            Height = CellSize / 2,
            Fill = color,
            Margin = new Thickness(x * CellSize + CellSize / 4, 
                                 y * CellSize + CellSize / 4, 0, 0)
        };
        MazeCanvas.Children.Add(ellipse);
    }

    private void DrawPath()
    {
        var pathColor = Brushes.Blue;
        foreach (var point in _path)
        {
            var x = (point.X + 0.5) * CellSize;
            var y = (point.Y + 0.5) * CellSize;
            var dot = new Ellipse
            {
                Width = 4,
                Height = 4,
                Fill = pathColor,
                Margin = new Thickness(x - 2, y - 2, 0, 0)
            };
            MazeCanvas.Children.Add(dot);
        }
    }

    private void BtnGenerate_Click(object sender, RoutedEventArgs e)
    {
        var generator = new MazeGenerator();
        _maze = generator.Generate(15, 15);
        _path.Clear();
        DrawMaze();
    }

    private void BtnSolve_Click(object sender, RoutedEventArgs e)
    {
        if (_maze == null) return;
        var solver = new PathSolver();
        _path = solver.SolveBFS(_maze);
        DrawMaze();
        DrawPath();
    }
}

C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MazeGenerator.cs
using System;
using System.Collections.Generic;
using System.Windows;

namespace WpfApp1;

public struct Cell
{
    public bool North;
    public bool South;
    public bool East;
    public bool West;
    public bool Visited;
}

public class MazeGenerator
{
    private Cell[,] _maze = new Cell[0,0]; // 初始化
    private readonly Random _random = new();
    // 修改方向处理
    private readonly (int dx, int dy)[] _directions = 
    {
        (0, -1),  // North
        (0, 1),   // South
        (1, 0),   // East
        (-1, 0)   // West
    };

    public Cell[,] Generate(int width, int height)
    {
        _maze = new Cell[height, width];
        InitializeCells();
        GenerateWithBacktracking();
        return _maze;
    }

    private void InitializeCells()
    {
        for (int y = 0; y < _maze.GetLength(0); y++)
        for (int x = 0; x < _maze.GetLength(1); x++)
            _maze[y, x] = new Cell
            {
                North = true, South = true, 
                East = true, West = true,
                Visited = false
            };
    }

    private void GenerateWithBacktracking()
    {
        var stack = new Stack<(int x, int y)>();
        stack.Push((0, 0));
        _maze[0, 0].Visited = true;

        var directions = new[]
        {
            new Point(0, -1),  // North
            new Point(0, 1),   // South
            new Point(1, 0),   // East
            new Point(-1, 0)   // West
        };

        while (stack.Count > 0)
        {
            var (x, y) = stack.Pop();
            
            var validDirs = _directions
                .Select((d, i) => new { Direction = d, Index = i })
                .Where(d => 
                {
                    var (dx, dy) = d.Direction;
                    int nx = x + dx;
                    int ny = y + dy;
                    return IsValidCell(nx, ny) && !_maze[ny, nx].Visited;
                })
                .ToList();

            if (validDirs.Count > 0)
            {
                var selected = validDirs[_random.Next(validDirs.Count)];
                var (dx, dy) = selected.Direction;
                int nx = x + dx;
                int ny = y + dy;

                RemoveWalls(x, y, selected.Index);
                _maze[ny, nx].Visited = true;
                stack.Push((x, y));
                stack.Push((nx, ny));
            }
        }
    }

    private void RemoveWalls(int x, int y, int dir)
    {
        switch (dir)
        {
            case 0: // North
                _maze[y, x].North = false;
                _maze[y - 1, x].South = false;
                break;
            case 1: // South
                _maze[y, x].South = false;
                _maze[y + 1, x].North = false;
                break;
            case 2: // East
                _maze[y, x].East = false;
                _maze[y, x + 1].West = false;
                break;
            case 3: // West
                _maze[y, x].West = false;
                _maze[y, x - 1].East = false;
                break;
        }
    }

    private bool IsValidCell(int x, int y) => 
        x >= 0 && x < _maze.GetLength(1) && 
        y >= 0 && y < _maze.GetLength(0);
}

C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\PathSolver.cs
using System;
using System.Collections.Generic;
using System.Windows;

namespace WpfApp1;

public class PathSolver
{
     
    
    public List<Point> SolveBFS(Cell[,] maze)
    {
        int width = maze.GetLength(1);
        int height = maze.GetLength(0);
        var visited = new bool[height, width];
        var prev = new (int x, int y)[height, width];
        var queue = new Queue<(int x, int y)>();
        var directions = new[] { (0, -1), (0, 1), (1, 0), (-1, 0) };

        queue.Enqueue((0, 0));
        visited[0, 0] = true;

        while (queue.Count > 0)
        {
            var (x, y) = queue.Dequeue();
            if (x == width - 1 && y == height - 1)
                return ReconstructPath(prev);

            foreach (var (dx, dy) in directions)
            {
                int nx = x + dx;
                int ny = y + dy;

                if (nx < 0 || nx >= width || ny < 0 || ny >= height) continue;
                if (visited[ny, nx]) continue;
                
                if (!CanMove(maze[y, x], GetDirectionIndex(dx, dy))) continue;

                visited[ny, nx] = true;
                prev[ny, nx] = (x, y);
                queue.Enqueue((nx, ny));
            }
        }
        return new List<Point>();
    }

    
    

    private int GetDirectionIndex(int dx, int dy) => (dx, dy) switch
    {
        (0, -1) => 0,  // North
        (0, 1) => 1,   // South
        (1, 0) => 2,   // East
        (-1, 0) => 3,  // West
        _ => -1
    };



    

    private List<Point> ReconstructPath((int x, int y)[,] prev)
    {
        var path = new List<Point>();
        (int x, int y) current = (prev.GetLength(1) - 1, prev.GetLength(0) - 1);
        
        while (current.x != 0 || current.y != 0)
        {
            path.Add(new Point(current.x, current.y));
            current = prev[current.y, current.x];
        }
        path.Add(new Point(0, 0));
        path.Reverse();
        return path;
    }



    private bool CanMove(Cell cell, int dir) => dir switch
    {
        0 => !cell.North,
        1 => !cell.South,
        2 => !cell.East,
        3 => !cell.West,
        _ => false
    };

    private List<Point> ReconstructPath(Point[,] prev)
    {
        var path = new List<Point>();
        var current = new Point(prev.GetLength(1) - 1, prev.GetLength(0) - 1);
        
        while (current.X != 0 || current.Y != 0)
        {
            path.Add(current);
            current = prev[(int)current.Y, (int)current.X];
        }
        path.Add(new Point(0, 0));
        path.Reverse();
        return path;
    }
}

C:\Users\wangrusheng\RiderProjects\WpfApp1\WpfApp1\MainWindow.xaml

<Window x:Class="WpfApp1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Maze Generator" Height="600" Width="800">
    <DockPanel>
        <StackPanel DockPanel.Dock="Top" Orientation="Horizontal" Margin="5">
            <Button x:Name="BtnGenerate" Content="Generate Maze" 
                    Click="BtnGenerate_Click" Margin="5" Padding="10 3"/>
            <Button x:Name="BtnSolve" Content="Solve Maze" 
                    Click="BtnSolve_Click" Margin="5" Padding="10 3"/>
        </StackPanel>
        <Canvas x:Name="MazeCanvas" Background="White" 
                Margin="10" ClipToBounds="True"/>
    </DockPanel>
</Window>



```

step6:纯算法

```cpp
C:\Users\wangrusheng\CLionProjects\untitled22\CMakeLists.txt

cmake_minimum_required(VERSION 3.30)
project(untitled22)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled22 main.cpp)


C:\Users\wangrusheng\CLionProjects\untitled22\main.cpp

#include <iostream>
#include <vector>
#include <stack>
#include <queue>
#include <algorithm>
#include <ctime>

using namespace std;

struct Cell {
    bool north;
    bool south;
    bool east;
    bool west;
    bool visited;
};

vector<vector<Cell>> maze;

// 生成迷宫
void generateMaze(int width, int height) {
    maze.resize(height, vector<Cell>(width));
    for (int y = 0; y < height; ++y) {
        for (int x = 0; x < width; ++x) {
            maze[y][x] = {true, true, true, true, false};
        }
    }

    srand(time(0));

    stack<pair<int, int>> stk;
    stk.push({0, 0});
    maze[0][0].visited = true;

    vector<pair<int, int>> dirs = {{0, -1}, {0, 1}, {1, 0}, {-1, 0}};

    while (!stk.empty()) {
        int x = stk.top().first;
        int y = stk.top().second;

        vector<int> possibleDirs;
        for (int i = 0; i < 4; ++i) {
            int nx = x + dirs[i].first;
            int ny = y + dirs[i].second;
            if (nx >= 0 && nx < width && ny >= 0 && ny < height && !maze[ny][nx].visited) {
                possibleDirs.push_back(i);
            }
        }

        if (!possibleDirs.empty()) {
            int dir = possibleDirs[rand() % possibleDirs.size()];
            int nx = x + dirs[dir].first;
            int ny = y + dirs[dir].second;

            switch (dir) {
                case 0: // North
                    maze[y][x].north = false;
                    maze[ny][nx].south = false;
                    break;
                case 1: // South
                    maze[y][x].south = false;
                    maze[ny][nx].north = false;
                    break;
                case 2: // East
                    maze[y][x].east = false;
                    maze[ny][nx].west = false;
                    break;
                case 3: // West
                    maze[y][x].west = false;
                    maze[ny][nx].east = false;
                    break;
            }

            maze[ny][nx].visited = true;
            stk.push({nx, ny});
        } else {
            stk.pop();
        }
    }
}

// 创建字符网格表示迷宫
vector<vector<char>> createGrid(int width, int height) {
    int gridWidth = 2 * width + 1;
    int gridHeight = 2 * height + 1;
    vector<vector<char>> grid(gridHeight, vector<char>(gridWidth, '#'));

    for (int y = 0; y < height; ++y) {
        for (int x = 0; x < width; ++x) {
            int cx = 2 * x + 1;
            int cy = 2 * y + 1;
            grid[cy][cx] = ' ';

            if (!maze[y][x].north) grid[cy - 1][cx] = ' ';
            if (!maze[y][x].south) grid[cy + 1][cx] = ' ';
            if (!maze[y][x].east) grid[cy][cx + 1] = ' ';
            if (!maze[y][x].west) grid[cy][cx - 1] = ' ';
        }
    }

    grid[1][1] = 'S';
    grid[gridHeight - 2][gridWidth - 2] = 'E';
    return grid;
}

// 定义点结构体用于路径追踪
struct Point {
    int x, y;
};

// BFS求解最短路径
bool solveMazeBFS(int startX, int startY, int endX, int endY, vector<Point>& path) {
    int width = maze[0].size();
    int height = maze.size();

    vector<vector<bool>> visited(height, vector<bool>(width, false));
    vector<vector<Point>> prev(height, vector<Point>(width, {-1, -1}));
    queue<Point> q;

    q.push({startX, startY});
    visited[startY][startX] = true;

    vector<pair<int, int>> dirs = {{0, -1}, {0, 1}, {1, 0}, {-1, 0}};

    while (!q.empty()) {
        Point curr = q.front();
        q.pop();

        if (curr.x == endX && curr.y == endY) {
            // 回溯路径
            Point p = curr;
            while (p.x != -1 || p.y != -1) {
                path.push_back(p);
                p = prev[p.y][p.x];
            }
            reverse(path.begin(), path.end());
            return true;
        }

        for (int i = 0; i < 4; ++i) {
            int nx = curr.x + dirs[i].first;
            int ny = curr.y + dirs[i].second;

            if (nx < 0 || nx >= width || ny < 0 || ny >= height) continue;

            bool canMove = false;
            switch (i) {
                case 0: canMove = !maze[curr.y][curr.x].north; break;
                case 1: canMove = !maze[curr.y][curr.x].south; break;
                case 2: canMove = !maze[curr.y][curr.x].east; break;
                case 3: canMove = !maze[curr.y][curr.x].west; break;
            }

            if (canMove && !visited[ny][nx]) {
                visited[ny][nx] = true;
                prev[ny][nx] = curr;
                q.push({nx, ny});
            }
        }
    }
    return false;
}

// 打印网格
void printGrid(const vector<vector<char>>& grid) {
    for (const auto& row : grid) {
        for (char c : row) {
            cout << c;
        }
        cout << endl;
    }
}

int main() {
    int width = 5, height = 5;
    generateMaze(width, height);
    auto grid = createGrid(width, height);

    cout << "Generated Maze:" << endl;
    printGrid(grid);

    vector<Point> path;
    if (solveMazeBFS(0, 0, width-1, height-1, path)) {
        cout << "\nPath Found:" << endl;
        // 标记路径
        for (const auto& p : path) {
            int cx = 2 * p.x + 1;
            int cy = 2 * p.y + 1;
            if (grid[cy][cx] != 'S' && grid[cy][cx] != 'E') {
                grid[cy][cx] = '.';
            }
        }
        printGrid(grid);
    } else {
        cout << "\nNo Path Found!" << endl;
    }

    return 0;
}
```

end