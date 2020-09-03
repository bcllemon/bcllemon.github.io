layout: post
title: "Spring Amqp Ack"
date: "2020-09-03 15:07"
categories: 技术
---
# Ack 模式说明
```
/**
 * No acks - {@code autoAck=true} in {@code Channel.basicConsume()}.
 */
NONE,

/**
 * Manual acks - user must ack/nack via a channel aware listener.
 */
MANUAL,

/**
 * Auto - the container will issue the ack/nack based on whether
 * the listener returns normally, or throws an exception.
 * <p><em>Do not confuse with RabbitMQ {@code autoAck} which is
 * represented by {@link #NONE} here</em>.
 */
AUTO;
```
# 代码逻辑
```
org.springframework.amqp.rabbit.listener.BlockingQueueConsumer#commitIfNecessary

public boolean commitIfNecessary(boolean localTx) throws IOException {

		if (this.deliveryTags.isEmpty()) {
			return false;
		}

		/*
		 * If we have a TX Manager, but no TX, act like we are locally transacted.
		 */
		boolean isLocallyTransacted = localTx
				|| (this.transactional
				&& TransactionSynchronizationManager.getResource(this.connectionFactory) == null);
		try {

			boolean ackRequired = !this.acknowledgeMode.isAutoAck() && !this.acknowledgeMode.isManual();

			if (ackRequired && (!this.transactional || isLocallyTransacted)) {
				long deliveryTag = new ArrayList<Long>(this.deliveryTags).get(this.deliveryTags.size() - 1);
				this.channel.basicAck(deliveryTag, true);
			}

			if (isLocallyTransacted) {
				// For manual acks we still need to commit
				RabbitUtils.commitIfNecessary(this.channel);
			}

		}
		finally {
			this.deliveryTags.clear();
		}

		return true;

	}
```