说明：
光线追踪渲染器
光线追踪是一种 模拟光线物理行为 的渲染技术，能够生成高度逼真的图像（如反射、折射、阴影等）。

光线投射：从相机发射光线到场景。
几何体相交检测：计算光线与球体的交点。
材质模拟：处理玻璃的折射（如菲涅耳效应）和漫反射。
阴影计算：通过光线遮挡判断生成阴影。
递归追踪：支持光线反射/折射的深度递归（MAX_RAY_DEPTH）。

交互式操作：通过按钮触发渲染，展示实时生成的图像。
物理效果模拟：
玻璃球的透明和折射效果（如光线穿过玻璃时的扭曲）。
地面和球体的漫反射（柔和的阴影过渡）。
光源的亮度影响（通过 EmissionColor 控制光源强度）。
动态调试：通过修改参数（如反射率、折射率），观察渲染结果的变化。
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1afaa2775ad7461cadc9d26d2fe1c5dd.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Vec3.cs

```csharp
using System;

namespace WinFormsApp3;

public class Vec3
{
    
 
        public float X, Y, Z;

        public Vec3(float x = 0, float y = 0, float z = 0)
        {
            X = x;
            Y = y;
            Z = z;
        }

        public Vec3 Normalize()
        {
            float len = Length;
            if (len > 0)
            {
                X /= len;
                Y /= len;
                Z /= len;
            }
            return this;
        }

        public float Dot(Vec3 v) => X * v.X + Y * v.Y + Z * v.Z;
        public float Length => (float)Math.Sqrt(X * X + Y * Y + Z * Z);

        public static Vec3 operator +(Vec3 a, Vec3 b) => new Vec3(a.X + b.X, a.Y + b.Y, a.Z + b.Z);
        public static Vec3 operator -(Vec3 a, Vec3 b) => new Vec3(a.X - b.X, a.Y - b.Y, a.Z - b.Z);
        public static Vec3 operator *(Vec3 a, float f) => new Vec3(a.X * f, a.Y * f, a.Z * f);
        public static Vec3 operator *(Vec3 a, Vec3 b) => new Vec3(a.X * b.X, a.Y * b.Y, a.Z * b.Z);
}
```

step2:C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Sphere.cs

```csharp
using System;

namespace WinFormsApp3;

public class Sphere
{
    
 
        public Vec3 Center;
        public float Radius, RadiusSq;
        public Vec3 SurfaceColor, EmissionColor;
        public float Reflection, Transparency, IOR;

        public Sphere(Vec3 center, float radius, Vec3 surfaceColor, 
            float reflection = 0, float transparency = 0, 
            Vec3 emissionColor = null, float ior = 1.5f)
        {
            Center = center;
            Radius = radius;
            RadiusSq = radius * radius;
            SurfaceColor = surfaceColor;
            Reflection = reflection;
            Transparency = transparency;
            EmissionColor = emissionColor ?? new Vec3(0);
            IOR = ior;
        }

        public bool Intersect(Vec3 rayOrig, Vec3 rayDir, out float t0, out float t1)
        {
            t0 = float.MaxValue;
            t1 = float.MaxValue;
            Vec3 l = Center - rayOrig;
            float tca = l.Dot(rayDir);
            if (tca < 0) return false;

            float d2 = l.Dot(l) - tca * tca;
            if (d2 > RadiusSq) return false;

            float thc = (float)Math.Sqrt(RadiusSq - d2);
            t0 = tca - thc;
            t1 = tca + thc;

            return true;
        }
}
```

step3:C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Form1.cs

```csharp
namespace WinFormsApp3;

public partial class Form1 : Form
{
    private Button btnRender;
    private PictureBox picRender;
    private Bitmap renderTarget;
    private List<Sphere> spheres = new List<Sphere>();
    private const int MAX_RAY_DEPTH = 5;
    private const float BIAS = 1e-4f;
    public Form1()
    {
        InitializeComponent();
    
        SetupForm();
    }
    
         private void SetupForm()
        {
            this.Text = "WinForm光线追踪器";
            this.Size = new Size(800, 600);

            btnRender = new Button
            {
                Text = "开始渲染",
                Location = new Point(10, 10)
            };
            btnRender.Click += BtnRender_Click;

            picRender = new PictureBox
            {
                Location = new Point(10, 50),
                Size = new Size(640, 480),
                BackColor = Color.Black
            };

            this.Controls.Add(btnRender);
            this.Controls.Add(picRender);
        }
 

        private void BtnRender_Click(object sender, EventArgs e)
        {
            InitializeScene();
            renderTarget = new Bitmap(picRender.Width, picRender.Height);
            picRender.Image = renderTarget;
            btnRender.Enabled = false;

            Task.Run(() => RenderScene());
        }

        private void InitializeScene()
        {
            spheres.Clear();
            // 地面
            spheres.Add(new Sphere(
                new Vec3(0, -10004, -20), 10000,
                new Vec3(0.2f, 0.2f, 0.2f), reflection: 0
            ));

            // 玻璃球
            spheres.Add(new Sphere(
                new Vec3(0, 0, -20), 4,
                new Vec3(1, 0.32f, 0.36f), 
                reflection: 0.9f, 
                transparency: 0.9f,
                ior: 1.5f
            ));

            // 光源
            spheres.Add(new Sphere(
                new Vec3(0, 20, -30), 3,
                new Vec3(0), 
                emissionColor: new Vec3(1.5f)
            ));
        }

        private Vec3 Trace(Vec3 rayOrig, Vec3 rayDir, List<Sphere> spheres, int depth)
        {
            float tnear = float.MaxValue;
            Sphere closestSphere = null;

            foreach (var sphere in spheres)
            {
                float t0, t1;
                if (sphere.Intersect(rayOrig, rayDir, out t0, out t1))
                {
                    if (t0 < 0) t0 = t1;
                    if (t0 < tnear)
                    {
                        tnear = t0;
                        closestSphere = sphere;
                    }
                }
            }

            if (closestSphere == null) return new Vec3(0.2f); // 背景色

            Vec3 phit = rayOrig + rayDir * tnear;
            Vec3 nhit = phit - closestSphere.Center;
            nhit.Normalize();

            // 矫正法线方向
            bool inside = false;
            if (rayDir.Dot(nhit) > 0)
            {
                nhit = new Vec3(-nhit.X, -nhit.Y, -nhit.Z);
                inside = true;
            }

            Vec3 surfaceColor = new Vec3(0);
            if ((closestSphere.Transparency > 0 || closestSphere.Reflection > 0) && depth < MAX_RAY_DEPTH)
            {
                float facingRatio = -rayDir.Dot(nhit);
                float fresnel = Mix(Math.Pow(1 - facingRatio, 3), 1, 0.1f);

                // 反射
                Vec3 refldir = rayDir - nhit * 2 * rayDir.Dot(nhit);
                refldir.Normalize();
                Vec3 reflection = Trace(phit + nhit * BIAS, refldir, spheres, depth + 1);

                // 折射
                Vec3 refraction = new Vec3(0);
                if (closestSphere.Transparency > 0)
                {
                    float ior = closestSphere.IOR;
                    float eta = inside ? ior : 1 / ior;
                    float cosi = -nhit.Dot(rayDir);
                    float k = 1 - eta * eta * (1 - cosi * cosi);

                    if (k >= 0)
                    {
                        Vec3 refrdir = rayDir * eta + nhit * (eta * cosi - (float)Math.Sqrt(k));
                        refrdir.Normalize();
                        refraction = Trace(phit - nhit * BIAS, refrdir, spheres, depth + 1);
                    }
                }

                surfaceColor = (reflection * fresnel + refraction * (1 - fresnel) * closestSphere.Transparency) * closestSphere.SurfaceColor;
            }
            else
            {
                // 漫反射
                foreach (var light in spheres)
                {
                    if (light.EmissionColor.X <= 0) continue;

                    Vec3 lightDir = light.Center - phit;
                    lightDir.Normalize();

                    // 阴影检测
                    bool inShadow = false;
                    foreach (var s in spheres)
                    {
                        if (s == light) continue;
                        float t0, t1;
                        if (s.Intersect(phit + nhit * BIAS, lightDir, out t0, out t1))
                        {
                            inShadow = true;
                            break;
                        }
                    }

                    if (!inShadow)
                    {
                        float lambert = Math.Max(0, nhit.Dot(lightDir));
                        surfaceColor += closestSphere.SurfaceColor * lambert * light.EmissionColor;
                    }
                }
            }

            return surfaceColor + closestSphere.EmissionColor;
        }

        private float Mix(double a, double b, double mix) => (float)(b * mix + a * (1 - mix));

        private void RenderScene()
        {
            int width = picRender.Width;
            int height = picRender.Height;
            float invWidth = 1.0f / width;
            float invHeight = 1.0f / height;
            float fov = 30;
            float aspect = width / (float)height;
            float angle = (float)Math.Tan(Math.PI * 0.5 * fov / 180);

            Parallel.For(0, height, y =>
            {
                for (int x = 0; x < width; x++)
                {
                    float xx = (2 * ((x + 0.5f) * invWidth) - 1) * angle * aspect;
                    float yy = (1 - 2 * ((y + 0.5f) * invHeight)) * angle;
                    Vec3 raydir = new Vec3(xx, yy, -1).Normalize();

                    Vec3 color = Trace(new Vec3(0), raydir, spheres, 0);

                    // 限制颜色范围并转换
                    color.X = Math.Max(0, Math.Min(1, color.X));
                    color.Y = Math.Max(0, Math.Min(1, color.Y));
                    color.Z = Math.Max(0, Math.Min(1, color.Z));

                    Color pixelColor = Color.FromArgb(
                        (int)(color.X * 255),
                        (int)(color.Y * 255),
                        (int)(color.Z * 255)
                    );

                    // 更新UI
                    this.Invoke((Action)(() => 
                    {
                        renderTarget.SetPixel(x, y, pixelColor);
                        picRender.Refresh();
                    }));
                }
            });

            this.Invoke((Action)(() => btnRender.Enabled = true));
        }
}
```

end