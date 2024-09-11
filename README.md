
### 【写在前面】


在很多工作中，我们需要计算数据或者文件的散列值，例如登录或下载文件。


而在 Qt 中，负责这项工作的类为 `QCryptographicHash`。


关于 `QCryptographicHash`：



> QCryptographicHash 是 Qt 框架中提供的一个用于生成加密散列（哈希值）的类。该类可以将任意长度的输入（二进制或文本数据）转换成固定长度的输出（哈希值），这一过程是不可逆的。`QCryptographicHash` 支持多种哈希算法，包括 `MD4、MD5、SHA-1、SHA-224、SHA-256、SHA-384 和 SHA-512` 等，这些算法在数据完整性校验、密码存储、数字签名等应用场景中非常有用。
> 
> 
> **主要特点：**
> 
> 
> 1. 支持多种哈希算法：`QCryptographicHash` 提供了多种哈希算法的支持，允许开发者根据具体需求选择合适的算法。
> 2. 简单易用的接口：`QCryptographicHash` 提供了简单易用的接口来计算哈希值。开发者可以通过调用 `QCryptographicHash::hash()` 静态方法或创建 `QCryptographicHash` 对象并使用 `addData()` 和 `result()` 方法来计算哈希值。
> 3. 逐块计算：\`QCryptographicHash 还可以逐块地计算哈希值，这对于处理大文件或流式数据非常有用。
> 4. 可重复使用：`QCryptographicHash` 对象可以多次使用。当计算完一个哈希值后，可以通过调用 `reset()` 方法重置对象，然后继续计算新的哈希值。


然鹅， 虽然 `QCryptographicHash` 很优秀，但它最大的问题在于其散列值的计算是同步的( 即阻塞 )，对小数据来说并没什么影响，但对大数据来说则意味明显卡顿。


因此，我将 `QCryptographicHash` 进行简单封装，扩展了实用性的同时并将计算改为异步，还增加了进度通知和结束通知。




---


### 【正文开始】


先来看看 `AsyncHasher` 的使用效果图：


![image](https://img2024.cnblogs.com/blog/802097/202409/802097-20240911101050124-2106295830.gif)


`AsyncHasher` 的使用方法非常简单：


1. 包含头文件：在使用 `AsyncHasher` 之前，需要包含相应的头文件 `#include "asynchasher.h"`。
2. 通过 `setSource / setSourceText / setSourceData/ setSourceObject` 设置源目标。
3. 通过 `void hashProgress(qint64 processed, qint64 total)` 来获取进度，`void finished()` 通知计算结束。
4. 通过 `QString hashValue() const` 获取最终结果。


例如 C\+\+ 使用：



```
    AsyncHasher *hasher = new AsyncHasher;
    hasher->setSourceText("Test Text");
    QObject::connect(hasher, &AsyncHasher::finished, [hasher]{
        qDebug() << hasher->hashValue();
    });

```

并且我还做了 Qml 适配，使用方法：



```
    AsyncHasher {
        id: textHasher
        algorithm: AsyncHasher.Md5
        onStarted: {
            startTime = Date.now();
        }
        onFinished: {
            totalTime = Date.now() - startTime;
            console.log("HashValue:", hashValue, "time:", totalTime);
        }
        property real startTime: 0
        property real totalTime: 0
    }

```

完整头文件如下：



```
#ifndef ASYNCHASHER_H
#define ASYNCHASHER_H

#include 
#include 
#include 
#include 

QT_FORWARD_DECLARE_CLASS(QNetworkAccessManager);

QT_FORWARD_DECLARE_CLASS(AsyncHasherPrivate);

class AsyncHasher : public QObject
{
    Q_OBJECT

    Q_PROPERTY(QCryptographicHash::Algorithm algorithm READ algorithm WRITE setAlgorithm NOTIFY algorithmChanged)
    Q_PROPERTY(bool asynchronous READ asynchronous WRITE setAsynchronous NOTIFY asynchronousChanged)
    Q_PROPERTY(QString hashValue READ hashValue NOTIFY hashValueChanged)
    Q_PROPERTY(int hashLength READ hashLength NOTIFY hashLengthChanged)
    Q_PROPERTY(QUrl source READ source WRITE setSource NOTIFY sourceChanged)
    Q_PROPERTY(QString sourceText READ sourceText WRITE setSourceText NOTIFY sourceTextChanged)
    Q_PROPERTY(QByteArray sourceData READ sourceData WRITE setSourceData NOTIFY sourceDataChanged)
    Q_PROPERTY(QObject* sourceObject READ sourceObject WRITE setSourceObject NOTIFY sourceObjectChanged)

public:
    Q_ENUMS(QCryptographicHash::Algorithm);

    explicit AsyncHasher(QObject *parent = nullptr);
    ~AsyncHasher();

    QNetworkAccessManager *networkManager() const;

    QCryptographicHash::Algorithm algorithm();
    void setAlgorithm(QCryptographicHash::Algorithm algorithm);

    bool asynchronous() const;
    void setAsynchronous(bool async);

    QString hashValue() const;

    int hashLength() const;

    QUrl source() const;
    void setSource(const QUrl &source);

    QString sourceText() const;
    void setSourceText(const QString &sourceText);

    QByteArray sourceData() const;
    void setSourceData(const QByteArray &sourceData);

    QObject *sourceObject() const;
    void setSourceObject(QObject *sourceObject);

    bool operator==(const AsyncHasher &hasher);
    bool operator!=(const AsyncHasher &hasher);

    QFuture static hash(const QByteArray &data, QCryptographicHash::Algorithm algorithm);

signals:
    void algorithmChanged();
    void asynchronousChanged();
    void hashValueChanged();
    void hashLengthChanged();
    void sourceChanged();
    void sourceTextChanged();
    void sourceDataChanged();
    void sourceObjectChanged();
    void hashProgress(qint64 processed, qint64 total);
    void started();
    void finished();

private slots:
    void setHashValue(const QString &value);

private:
    Q_DECLARE_PRIVATE(AsyncHasher);
    QScopedPointer d_ptr;
};

#endif // ASYNCHASHER_H

```



---


### 【结语】


最后：项目链接(多多star呀..⭐\_⭐)：


Github 地址：[https://github.com/mengps/QmlControls/tree/master/AsyncHasher](https://github.com)


 ![](https://github.com/blog/802097/202404/802097-20240417231514692-194340162.png)    - **本文作者：** [梦起丶](https://github.com)
 - **本文链接：** [https://github.com/mengps/p/18407761](https://github.com):[MeoMiao 萌喵加速](https://biqumo.org)
 - **关于博主：** 🎉🎉🎉专注于 C / C\+\+ / Qt / JS / Python 编程技巧🎉🎉🎉

✨个人微信✨：MenPenS612 (交流合作都行\~)

✨交流群✨：83986890 ヾ(￣▽￣)欢迎来玩\~

✨公众号✨：程序梦 (扫左侧二维码)
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
