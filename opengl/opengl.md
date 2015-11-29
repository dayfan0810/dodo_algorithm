
MyGLSurfaceView.java
```java
public class MyGLSurfaceView extends GLSurfaceView {
    public MyGLSurfaceView(Context context) {
        super(context);
        initView();
    }


    private void initView() {
        setEGLContextClientVersion(2);
        MultisampleConfigChooser eglConfigChooser = new MultisampleConfigChooser();
        setEGLConfigChooser(eglConfigChooser);
        setRenderer(new MyRender());
        setRenderMode(RENDERMODE_WHEN_DIRTY);
    }
}

```


MyRender.java
```java

package com.example.baidu.opengl;

import android.opengl.GLES20;
import android.opengl.GLSurfaceView;
import android.util.Log;

import javax.microedition.khronos.egl.EGLConfig;
import javax.microedition.khronos.opengles.GL10;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.nio.FloatBuffer;

/**
 * Created by baidu on 15/9/15.
 */
public class MyRender implements GLSurfaceView.Renderer {

    private Triangle mTriangle;

    @Override public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        GLES20.glClearColor(0.5f, 0.5f, 0.5f, 1f);//清空当前的所有颜色
        mTriangle = new Triangle();
    }

    @Override public void onSurfaceChanged(GL10 gl, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
        float ratio = (float) width / height;
        MatrixState.setProjectOrtho(-ratio, ratio, -1f, 1f, 1, 10);
        MatrixState.setCamera(0.0f, 0.0f, 3.0f, 0.0f, 0.0f, 0.0f, 0.0f, 1.0f, 0.0f);
        MatrixState.initMatrix();
    }

    @Override public void onDrawFrame(GL10 gl) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);//清除颜色缓冲
        mTriangle.draw2();
    }

    /**
     * 绘制一个形状
     * 使用OpenGL ES 2.0绘制一个定义好的形状需要大量的代码，因为必须提供给图形渲染管道很多细节信息。具体定义如下：
     * 1VertexShader：用于呈现形状顶点的OpenGL ES图形代码。
     * 2FragmentShader：用于呈现形状外观（颜色或纹理）的OpenGL ES代码。
     * 3Program：一个OpenGL ES对象，包含了你想要用来绘制一个或多个形状的shader。
     * 你至少需要一个vertex shader来绘制一个形状和一个fragment shader来为形状着色。这些shader必须被编译，然后将它们添加到一个OpenGL ES program中，接着使用program绘制形状。
     **/

    /**
     * 用于呈现顶点形状的ShaderCode
     * 
     */
    private final String vertexShaderCode = "uniform mat4 uMVPMatrix; " +
            "attribute vec3 aPosition;" +
            "attribute vec2 aTexCoor; " +
            "varying vec2 vTextureCoord;  " +
            "attribute vec4 aColor;  " +
            "varying  vec4 vColor;  " +
            "void main()     " +
            "{" +
            "   gl_Position = uMVPMatrix * vec4(aPosition,1); " +
            "   vTextureCoord = aTexCoor;" +
            "   gl_PointSize=10.0;" +
            "   vColor = aColor;" +
            "}";
    private final String fragmentShaderCode = "precision mediump float;" +
            "uniform int uIsColorFrag;" +
            "uniform sampler2D sTexture;" +
            "varying vec2 vTextureCoord;" +
            "varying  vec4 vColor;" +
            "void main()" +
            "{if(uIsColorFrag==1)" +
            "{gl_FragColor = vColor;}" +
            "else{gl_FragColor = texture2D(sTexture, vTextureCoord); }}";

    //    private final String vertexShaderCode = "attribute vec4 vPosition;" +
    //            "void main() {" +
    //            "  gl_Position = vPosition;" +
    //            "}";
    //
    //    private final String fragmentShaderCode = "precision mediump float;" +
    //            "uniform vec4 vColor;" +
    //            "void main() {" +
    //            "  gl_FragColor = vColor;" +
    //            "}";

    private int loadShader(int type, String shaderCode) {

        // 创建一个vertex shader类型(GLES20.GL_VERTEX_SHADER)
        // 或一个fragment shader类型(GLES20.GL_FRAGMENT_SHADER)
        int shader = GLES20.glCreateShader(type);

        // 将源码添加到shader并编译它
        GLES20.glShaderSource(shader, shaderCode);
        GLES20.glCompileShader(shader);

        return shader;
    }

    class Triangle {
        private final String TAG = Triangle.class.getSimpleName();
        private FloatBuffer vertexBuffer;//顶点的buffer
        private FloatBuffer colorsBuffer;//颜色的buffer

        // 设置每个顶点的坐标数
        static final int COORDS_PER_VERTEX = 3;
        // 设置三角形顶点数组
        final float triangleCoords[] = { // 默认按逆时针方向顺序绘制
                0.0f, 0.622008459f, 0.0f,   // 顶
                -0.5f, -0.311004243f, 0.0f,   // 左底
                0.5f, -0.311004243f, 0.0f    // 右底
        };

        // 设置图形的RGB值和透明度
        float color[] =
                { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f, 0.63671875f, 0.76953125f, 0.22265625f, 1.0f, 0.63671875f,
                        0.76953125f, 0.22265625f, 1.0f, 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };
        private int mPositionHandle;
        private int mColorHandle;
        /**
         * 总变换矩阵位置
         */
        protected int mUMvpMatrixLocation;
        /**
         * 顶点位置
         */
        protected int mAPositionLocation;
        /**
         * 顶点颜色位置
         */
        protected int mAColorLocation;
        protected int mProgram;
        /**
         * 纹理的参数
         */
        protected int isColorFrag;
        /**
         * 纹理
         */
        protected int mATexCoor;

        public Triangle() {
            initShader();
            // 初始化顶点字节缓冲区，用于存放形状的坐标，
            ByteBuffer bb = ByteBuffer.allocateDirect(
                    // (每个浮点数占用4个字节)
                    triangleCoords.length * 4);
            // 设置使用设备硬件的原生字节序
            bb.order(ByteOrder.nativeOrder());

            // 从ByteBuffer中创建一个浮点缓冲区
            vertexBuffer = bb.asFloatBuffer();
            // 把坐标都添加到FloatBuffer中
            vertexBuffer.put(triangleCoords);
            // 设置buffer从第一个坐标开始读
            vertexBuffer.position(0);

            ByteBuffer bBuffer = ByteBuffer.allocateDirect(color.length * 4);
            bBuffer.order(ByteOrder.nativeOrder());
            colorsBuffer = bBuffer.asFloatBuffer();
            colorsBuffer.clear();
            colorsBuffer.put(color);
            colorsBuffer.position(0);
        }

        private void initShader() {
            int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, vertexShaderCode);
            int fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, fragmentShaderCode);

            mProgram = GLES20.glCreateProgram();             // 创建空的OpenGL ES Program
            GLES20.glAttachShader(mProgram, vertexShader);   // 将vertex shader添加到program
            GLES20.glAttachShader(mProgram, fragmentShader); // 将fragment shader添加到program
            GLES20.glLinkProgram(mProgram);                  // 创建可执行的 OpenGL ES program
            //存放链接成功program数量的数组

            int[] linkStatus = new int[1];

            //获取program的链接情况

            GLES20.glGetProgramiv(mProgram, GLES20.GL_LINK_STATUS, linkStatus, 0);

            //若链接失败则报错并删除程序

            if (linkStatus[0] != GLES20.GL_TRUE) {

                Log.e(TAG, "Could not link program: ");

                Log.e(TAG, GLES20.glGetProgramInfoLog(mProgram));

                GLES20.glDeleteProgram(mProgram);

                mProgram = 0;

            }

            mAPositionLocation = GLES20.glGetAttribLocation(mProgram, "aPosition");
            mAColorLocation = GLES20.glGetAttribLocation(mProgram, "aColor");
            mUMvpMatrixLocation = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");
            isColorFrag = GLES20.glGetUniformLocation(mProgram, "uIsColorFrag");
            mATexCoor = GLES20.glGetAttribLocation(mProgram, "aTexCoor");

        }

        public void draw2() {
            GLES20.glUseProgram(mProgram);
            GLES20.glUniformMatrix4fv(mUMvpMatrixLocation, 1, false, MatrixState.getFinalMatrix(), 0);
            GLES20.glVertexAttribPointer(mAPositionLocation, 3, GLES20.GL_FLOAT, false, 3 * 4, vertexBuffer);
            GLES20.glVertexAttribPointer(mAColorLocation, 4, GLES20.GL_FLOAT, false, 4 * 4, colorsBuffer);
            GLES20.glEnableVertexAttribArray(mAPositionLocation);
            GLES20.glEnableVertexAttribArray(mAColorLocation);
            GLES20.glLineWidth(1);
            GLES20.glDrawArrays(GLES20.GL_POINTS, 0, COORDS_PER_VERTEX);
            GLES20.glDisableVertexAttribArray(mAPositionLocation);
            GLES20.glDisableVertexAttribArray(mAColorLocation);
        }

        /**
         *
         */
        public void draw() {
            // 添加program到OpenGL ES环境中
            GLES20.glUseProgram(mProgram);

            // 获取指向vertex shader的成员vPosition的handle
            mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");

            // 启用一个指向三角形的顶点数组的handle
            GLES20.glEnableVertexAttribArray(mPositionHandle);

            // 准备三角形的坐标数据
            /**
             void glVertexAttribPointer (int index, int size, int type, boolean normalized, int stride, Buffer ptr )
             index  指定要修改的顶点着色器中顶点变量id；
             size   指定每个顶点属性的组件数量。必须为1、2、3或者4。如position是由3个（x,y,z）组成，而颜色是4个（r,g,b,a））；
             type   指定数组中每个组件的数据类型。可用的符号常量有GL_BYTE, GL_UNSIGNED_BYTE, GL_SHORT,GL_UNSIGNED_SHORT, GL_FIXED, 和 GL_FLOAT，初始值为GL_FLOAT；
             normalized  指定当被访问时，固定点数据值是否应该被归一化（GL_TRUE）或者直接转换为固定点值（GL_FALSE）；
             stride      指定连续顶点属性之间的偏移量。如果为0，那么顶点属性会被理解为：它们是紧密排列在一起的。初始值为0。如果normalized被设置为GL_TRUE，意味着整数型的值会被映射至区间[-1,1](有符号整数)，或者区间[0,1]（无符号整数），反之，这些值会被直接转换为浮点值而不进行归一化处理；
             ptr  顶点的缓冲数据。
             **/
            GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX, GLES20.GL_FLOAT, false, 0, vertexBuffer);

            // 获取指向fragment shader的成员vColor的handle
            mColorHandle = GLES20.glGetUniformLocation(mProgram, "vColor");

            // 设置三角形的颜色
            GLES20.glUniform4fv(mColorHandle, 1, color, 0);

            // 绘制三角形
            GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, COORDS_PER_VERTEX);

            // 禁用指向三角形的顶点数组
            GLES20.glDisableVertexAttribArray(mPositionHandle);
        }
    }
}


```