---
title: notes on pytorch
---

### basic concepts:

model (nn.Module的子类) 的参数保存在
`model.state_dict()`, 这个 OrderedDict 是该model参数的name-value dict
`model.state_dict().keys()` 可以查看所有参数的名称

`torch.save(model.state_dict(), "model.pth")` 将该model的参数dict 保存到文件中

我们可以通过 `d = torch.load("model.pth")` 来加载查看这个dict中的name和value, 原则上，我们修改这个dict中的name 和 value

`model.load_state_dict(d, strict=False)` 将dict d 保存的值 按照name关系，赋值给 model 相应的参数


```
class CustomDataset(Dataset):
  def __getitem__(self, idx)

dataset = CustomDataset(...)
dataloader = DataLoader(dataset, batch_size=64, shuffle=True)

class model(nn.Module):
  def forward

model.to(device)

optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)
pred=model(X)
loss_fn = nn.CrossEntropyLoss()
loss=loss_fn(pred,y)

loss.backward() #calc dirative

optimizer.step() 

torch.save(model.state_dict(), "model.pth")
model.load_state_dict(torch.load("model.pth"))

```


### grad API

https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html

w = torch.randn(5, 3, requires_grad=True) or

w = torch.randn(5, 3)
w.requires_grad_(True)

every nodes has t.grad_fn
leaf nodes has w.grad

We can only obtain the grad properties for the leaf nodes of the computational graph, which have requires_grad property set to True. For all other nodes in our graph, gradients will not be available.


There are reasons you might want to disable gradient tracking:
To mark some parameters in your neural network as frozen parameters. This is a very common scenario for finetuning a pretrained network

To speed up computations when you are only doing forward pass, because computations on tensors that do not track gradients would be more efficient.


By default, all tensors with requires_grad=True are tracking their computational history and support gradient computation. However, there are some cases when we do not need to do that, for example, when we have trained the model and just want to apply it to some input data, i.e. we only want to do forward computations through the network. We can stop tracking computations by surrounding our computation code with torch.no_grad() block:

### sd codes analysis

- yaml defines model structure 
- ckpt are a state_dict of component-parameter values 
- yaml and ckpt are pairwisely generated by trainer
- model.load_state_dict load into model with the parameter values in ckpt 

how to transform sd code to diffuser pipelines?

- initialize sd model from yaml and ckpt 
- find piece of vae, unet, tokenizer ... in model conponents
- iniatlize pipeline with vae, unet, ...
- pipeline.save_pretrained to get a pipeline model-directory
