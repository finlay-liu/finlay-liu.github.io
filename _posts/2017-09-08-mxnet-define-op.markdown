---
layout:     post
title:      "MXNet 定义新激活函数"
subtitle:   " \"Mxnet define OP\""
date:       2017-9-8 12:00:00
author:     "FinlayLiu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 深度学习
---

这里使用比较简单的定义方式，只是在原有的激活函数调用中加入。

- 准备工作

下载MXNet源代码，确认可以顺利编译通过。推荐在Linux下进行此操作:

https://mxnet.incubator.apache.org/get_started/install.html

- 编写激活函数先前和先后传递

在`src/operator/mshadow_op.h`里面，加入新的激活函数向前传递和向后的函数：

```
/*!
 * \brief RBF Unit
 * \author Yuzhong Liu
*/
struct rbf {
  template<typename DType>
  MSHADOW_XINLINE static DType Map(DType x) {
    return DType(expf(-x*x));
  }
};

struct rbf_grad {
  template<typename DType>
  MSHADOW_XINLINE static DType Map(DType x, DType a) {
    return DType(-2 * x * a);
  }
};
```
- 添加调用方法

在`src/operator/leaky_relu-inl.h`里面，激活函数的调用方式：

```
namespace leakyrelu {
enum LeakyReLUOpInputs {kData, kGamma};
enum LeakyReLUOpOutputs {kOut, kMask};
# 定义新的激活函数名称
enum LeakyReLUOpType {kLeakyReLU, kPReLU, kRReLU, kELU, kRBF};
enum LeakyReLUOpResource {kRandom};
}  // namespace leakyrelu

struct LeakyReLUParam : public dmlc::Parameter<LeakyReLUParam> {
  // use int for enumeration
  int act_type;
  float slope;
  float lower_bound;
  float upper_bound;
  DMLC_DECLARE_PARAMETER(LeakyReLUParam) {
    DMLC_DECLARE_FIELD(act_type).set_default(leakyrelu::kLeakyReLU)
    .add_enum("rrelu", leakyrelu::kRReLU)
    .add_enum("leaky", leakyrelu::kLeakyReLU)
    .add_enum("prelu", leakyrelu::kPReLU)
    .add_enum("elu", leakyrelu::kELU)
    # 添加激活函数枚举
    .add_enum("rbf", leakyrelu::kRBF)
    .describe("Activation function to be applied.");
    DMLC_DECLARE_FIELD(slope).set_default(0.25f)
    .describe("Init slope for the activation. (For leaky and elu only)");
    DMLC_DECLARE_FIELD(lower_bound).set_default(0.125f)
    .describe("Lower bound of random slope. (For rrelu only)");
    DMLC_DECLARE_FIELD(upper_bound).set_default(0.334f)
    .describe("Upper bound of random slope. (For rrelu only)");
  }
};

template<typename xpu>
class LeakyReLUOp : public Operator {
 public:
  explicit LeakyReLUOp(LeakyReLUParam param) {
    param_ = param;
  }

  virtual void Forward(const OpContext &ctx,
                       const std::vector<TBlob> &in_data,
                       const std::vector<OpReqType> &req,
                       const std::vector<TBlob> &out_data,
                       const std::vector<TBlob> &aux_args) {
    using namespace mshadow;
    using namespace mshadow::expr;
    size_t expected = param_.act_type == leakyrelu::kPReLU ? 2 : 1;
    CHECK_EQ(in_data.size(), expected);
    Stream<xpu> *s = ctx.get_stream<xpu>();
    Tensor<xpu, 3> data;
    Tensor<xpu, 3> out;
    Tensor<xpu, 3> mask;
    Tensor<xpu, 1> weight;
    int n = in_data[leakyrelu::kData].shape_[0];
    int k = in_data[leakyrelu::kData].shape_[1];
    Shape<3> dshape = Shape3(n, k, in_data[leakyrelu::kData].Size()/n/k);
    data = in_data[leakyrelu::kData].get_with_shape<xpu, 3, real_t>(dshape, s);
    out = out_data[leakyrelu::kOut].get_with_shape<xpu, 3, real_t>(dshape, s);
    if (param_.act_type == leakyrelu::kRReLU) {
      mask = out_data[leakyrelu::kMask].get_with_shape<xpu, 3, real_t>(dshape, s);
    }
    switch (param_.act_type) {
      case leakyrelu::kLeakyReLU: {
        Assign(out, req[leakyrelu::kOut], F<mshadow_op::xelu>(data, param_.slope));
        break;
      }
      case leakyrelu::kPReLU: {
        weight = in_data[leakyrelu::kGamma].get<xpu, 1, real_t>(s);
        Assign(out, req[leakyrelu::kOut],
               F<mshadow_op::xelu>(data, broadcast<1>(weight, out.shape_)));
        break;
      }
      case leakyrelu::kRReLU: {
        if (ctx.is_train) {
          Random<xpu>* prnd = ctx.requested[leakyrelu::kRandom].get_random<xpu, real_t>(s);
          mask = prnd->uniform(mask.shape_);
          mask = mask * (param_.upper_bound - param_.lower_bound) + param_.lower_bound;
          Assign(out, req[leakyrelu::kOut], F<mshadow_op::xelu>(data, mask));
        } else {
          const float slope = (param_.lower_bound + param_.upper_bound) / 2.0f;
          Assign(out, req[leakyrelu::kOut], F<mshadow_op::xelu>(data, slope));
        }
        break;
      }
      case leakyrelu::kELU: {
        Assign(out, req[leakyrelu::kOut], F<mshadow_op::elu>(data, param_.slope));
        break;
      }
      # RBF向前
      case leakyrelu::kRBF: {
        Assign(out, req[leakyrelu::kOut], F<mshadow_op::rbf>(data));
        break;
      }
      default:
        LOG(FATAL) << "Not implmented";
    }
  }

  virtual void Backward(const OpContext & ctx,
                        const std::vector<TBlob> &out_grad,
                        const std::vector<TBlob> &in_data,
                        const std::vector<TBlob> &out_data,
                        const std::vector<OpReqType> &req,
                        const std::vector<TBlob> &in_grad,
                        const std::vector<TBlob> &aux_args) {
    using namespace mshadow;
    using namespace mshadow::expr;
    size_t expected = param_.act_type == leakyrelu::kPReLU ? 2 : 1;
    CHECK_EQ(out_grad.size(), 1U);
    CHECK_EQ(req.size(), expected);
    CHECK_EQ(in_data.size(), expected);
    Stream<xpu> *s = ctx.get_stream<xpu>();
    Tensor<xpu, 3> output;
    Tensor<xpu, 3> data;
    Tensor<xpu, 3> gdata;
    Tensor<xpu, 3> grad;
    Tensor<xpu, 3> mask;
    Tensor<xpu, 1> weight;
    Tensor<xpu, 1> grad_weight;
    int n = out_grad[leakyrelu::kOut].shape_[0];
    int k = out_grad[leakyrelu::kOut].shape_[1];
    Shape<3> dshape = Shape3(n, k, out_grad[leakyrelu::kOut].Size()/n/k);
    grad = out_grad[leakyrelu::kOut].get_with_shape<xpu, 3, real_t>(dshape, s);
    gdata = in_grad[leakyrelu::kData].get_with_shape<xpu, 3, real_t>(dshape, s);
    output = out_data[leakyrelu::kOut].get_with_shape<xpu, 3, real_t>(dshape, s);
    if (param_.act_type == leakyrelu::kRReLU) {
      mask = out_data[leakyrelu::kMask].get_with_shape<xpu, 3, real_t>(dshape, s);
    }
    if (param_.act_type == leakyrelu::kPReLU) {
      data = in_data[leakyrelu::kData].get_with_shape<xpu, 3, real_t>(dshape, s);
    }
    switch (param_.act_type) {
      case leakyrelu::kLeakyReLU: {
        Assign(gdata, req[leakyrelu::kData], F<mshadow_op::xelu_grad>(output, param_.slope) * grad);
        break;
      }
      case leakyrelu::kPReLU: {
        weight = in_data[leakyrelu::kGamma].get<xpu, 1, real_t>(s);
        grad_weight = in_grad[leakyrelu::kGamma].get<xpu, 1, real_t>(s);
        grad_weight = sumall_except_dim<1>(F<prelu_grad>(data) * grad);
        gdata = F<mshadow_op::xelu_grad>(data, broadcast<1>(weight, data.shape_)) * grad;
        break;
      }
      case leakyrelu::kRReLU: {
        Assign(gdata, req[leakyrelu::kData], F<mshadow_op::xelu_grad>(output, mask) * grad);
        break;
      }
      case leakyrelu::kELU: {
        Assign(gdata, req[leakyrelu::kData], F<mshadow_op::elu_grad>(output, param_.slope) * grad);
        break;
      }
      # RBF向前
      case leakyrelu::kRBF: {
        data = in_data[leakyrelu::kData].get_with_shape<xpu, 3, real_t>(dshape, s);
        Assign(gdata, req[leakyrelu::kData], F<mshadow_op::rbf_grad>(data, output) * grad);
        break;
      }
      default:
        LOG(FATAL) << "Not implmented";
    }
  }

 private:
  LeakyReLUParam param_;
};  // class LeakyReLUOp
```

- 从重新编译，并测试


```
import mxnet as mx
from mxnet import autograd
a = mx.nd.random_uniform(-1, 1, shape=[3, 3]) +0.5
a.attach_grad()

with autograd.record():
    b = mx.nd.LeakyReLU(data=a, act_type='rbf')

print a, b
```

- 参考资料

https://mxnet.incubator.apache.org/how_to/new_op.html
http://blog.csdn.net/qq_20965753/article/details/66975622?utm_source=debugrun&utm_medium=referral
