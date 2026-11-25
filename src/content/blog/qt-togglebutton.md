---
title: "Qt ToggleButton"
description: "用 Qt 实现一个带滑动动画的 iOS 风格 ToggleButton 开关。"
pubDate: 2025-06-22
tags: ['Tech']
---

## 实现效果

<video autoplay muted loop playsinline preload="metadata" width="120" height="34" aria-label="ToggleButton 开关演示">
  <source src="/uploads/2025/06/QtToggleButton.mp4" type="video/mp4" />
  <img src="/uploads/2025/06/QtToggleButton.gif" alt="ToggleButton 开关演示" width="120" height="34" loading="lazy" />
</video>

## 头文件

```
#pragma once
#include <QWidget>

class QPropertyAnimation;
class ToggleButtonPrivate;
class ToggleButton : public QWidget
{
    Q_OBJECT
    Q_PROPERTY(bool checked READ isChecked WRITE setChecked NOTIFY toggled)
    Q_PROPERTY(qreal indicatorPos READ indicatorPos WRITE setIndicatorPos)

public:
    explicit ToggleButton(QWidget *parent = nullptr);

    void setOnText(const QString &text);
    void setOffText(const QString &text);

    bool isChecked() const;
    void setChecked(bool checked);

    qreal indicatorPos() const;
    void setIndicatorPos(qreal pos);
    void setIndicatorWidth(int w);
    int indicatorWidth() const;

Q_SIGNALS:
    void toggled(bool checked);

protected:
    void paintEvent(QPaintEvent *event) override;
    void mousePressEvent(QMouseEvent *event) override;
    QSize sizeHint() const override;

private:
    ToggleButtonPrivate *d_ptr;
    Q_DECLARE_PRIVATE(ToggleButton);
};
```

## 实现代码

```
#include <QPainter>
#include <QMouseEvent>
#include <QEasingCurve>

#include <QPainter>
#include <QPropertyAnimation>
#include <QMouseEvent>

constexpr int kTextTrackSpacing = 4;

class ToggleButtonPrivate
{
public:
    int indicatorWidth = 48; // 默认滑动轨道宽度
    bool checked = false;
    qreal indicatorPos = 0; // 0=off, 1=on
    QString onText{};
    QString offText{};
    QPropertyAnimation *anim = nullptr;
    void startToggleAnimation(bool checked)
    {
        anim->stop();
        anim->setStartValue(indicatorPos);
        anim->setEndValue(checked ? 1.0 : 0.0);
        anim->start();
    }
};

ToggleButton::ToggleButton(QWidget *parent)
    : QWidget(parent)
    , d_ptr(new ToggleButtonPrivate)
{
    setCursor(Qt::PointingHandCursor);
    setMinimumHeight(32);
    setMinimumWidth(60);

    Q_D(ToggleButton);
    d->anim = new QPropertyAnimation(this, "indicatorPos", this);
    d->anim->setDuration(150);
    d->anim->setEasingCurve(QEasingCurve::InOutCubic);

    setChecked(false);
}

void ToggleButton::setOnText(const QString &text)
{
    Q_D(ToggleButton);
    d->onText = text;
    update();
}
void ToggleButton::setOffText(const QString &text)
{
    Q_D(ToggleButton);
    d->offText = text;
    update();
}
bool ToggleButton::isChecked() const
{
    Q_D(const ToggleButton);
    return d->checked;
}

void ToggleButton::setChecked(bool checked)
{
    Q_D(ToggleButton);
    if (d->checked == checked) {
        return;
    }
    d->checked = checked;
    d->startToggleAnimation(checked);
    emit toggled(checked);
}

qreal ToggleButton::indicatorPos() const
{
    Q_D(const ToggleButton);
    return d->indicatorPos;
}

void ToggleButton::setIndicatorPos(qreal pos)
{
    Q_D(ToggleButton);
    d->indicatorPos = pos;
    update();
}

void ToggleButton::setIndicatorWidth(int w)
{
    Q_D(ToggleButton);
    if (d->indicatorWidth != w) {
        d->indicatorWidth = w;
        update();
    }
}
int ToggleButton::indicatorWidth() const
{
    Q_D(const ToggleButton);
    return d->indicatorWidth;
}

void ToggleButton::paintEvent(QPaintEvent *)
{
    Q_D(ToggleButton);
    QPainter p(this);
    p.setRenderHint(QPainter::Antialiasing, true);

    QFontMetrics fm(font());
    int offTextWidth = fm.horizontalAdvance(d->offText);
    int onTextWidth = fm.horizontalAdvance(d->onText);

    // [OffText][2px][轨道][2px][OnText]
    int trackW = d->indicatorWidth;
    int trackH = height() * 0.8;
    trackH = qMin(trackH, height() - 4);
    int trackX = offTextWidth + kTextTrackSpacing;
    int trackY = (height() - trackH) / 2;
    QRectF trackRect(trackX, trackY, trackW, trackH);

    // 轨道
    QColor bg = d->checked ? QColor("#4CAF50") : QColor("#BDBDBD");
    p.setPen(Qt::NoPen);
    p.setBrush(bg);
    p.drawRoundedRect(trackRect, trackRect.height() / 2, trackRect.height() / 2);

    // knob
    qreal knobDiameter = trackRect.height() - 4;
    qreal knobY = trackRect.top() + 2;
    qreal left = trackRect.left() + 2;
    qreal right = trackRect.right() - knobDiameter - 2;
    qreal knobX = left + (right - left) * d->indicatorPos;
    p.setBrush(Qt::white);
    p.drawEllipse(QRectF(knobX, knobY, knobDiameter, knobDiameter));

    // OffText
    p.setPen(palette().color(QPalette::WindowText));
    p.drawText(QRectF(0, 0, offTextWidth, height()), Qt::AlignVCenter | Qt::AlignLeft, d->offText);

    // OnText
    p.drawText(QRectF(trackX + trackW + kTextTrackSpacing, 0, onTextWidth, height()),
               Qt::AlignVCenter | Qt::AlignLeft,
               d->onText);
}

void ToggleButton::mousePressEvent(QMouseEvent *event)
{
    Q_D(ToggleButton);
    if (event->button() == Qt::LeftButton) {
        setChecked(!d->checked);
    }
    QWidget::mousePressEvent(event);
}

QSize ToggleButton::sizeHint() const
{
    Q_D(const ToggleButton);
    QFontMetrics fm(font());
    int h = qMax((int) (fm.height() * 1.3), d->indicatorWidth / 2 + 4);
    int w = fm.horizontalAdvance(d->offText) + kTextTrackSpacing + d->indicatorWidth
            + kTextTrackSpacing + fm.horizontalAdvance(d->onText);
    return QSize(w, h);
}
```

## 使用方式

```
toggleBtn = new ToggleButton();
toggleBtn->setOnText("On");
toggleBtn->setOffText("Off");
connect(toggleBtn, &ToggleButton::toggled, [=](bool checked) {
    // ...
});
toggleBtn->setChecked(true);
```
