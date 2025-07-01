import React, { useState, useEffect } from 'react';
import { View, Text, Image, Button, StyleSheet } from 'react-native';
import io from 'socket.io-client';

const socket = io('http://tu-servidor.com');

const GameScreen = () => {
  const [cards, setCards] = useState([]);
  const [fortuneCards, setFortuneCards] = useState([]);
  const [phase, setPhase] = useState('start');

  useEffect(() => {
    // Conexión al servidor y escucha de eventos
    socket.on('dealCards', (data) => {
      setCards(data.playerCards);
      setFortuneCards(data.fortuneCards);
    });

    socket.on('phaseChange', (newPhase) => {
      setPhase(newPhase);
    });

    return () => {
      socket.off('dealCards');
      socket.off('phaseChange');
    };
  }, []);

  const handleExchange = (cardsToExchange) => {
    socket.emit('exchangeCards', cardsToExchange);
  };

  const handleBet = (amount) => {
    socket.emit('placeBet', amount);
  };

  const handleJackpot = () => {
    socket.emit('launchJackpot');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>JACKPOT</Text>
      <View style={styles.cardArea}>
        {cards.map((card, index) => (
          <Image key={index} source={{ uri: card.image }} style={styles.card} />
        ))}
      </View>
      <View style={styles.fortuneArea}>
        {fortuneCards.map((card, index) => (
          <Image key={index} source={{ uri: card.image }} style={styles.card} />
        ))}
      </View>
      <View style={styles.controls}>
        {phase === 'exchange' && (
          <Button title="Intercambiar Cartas" onPress={() => handleExchange([])} />
        )}
        {phase === 'bet' && (
          <Button title="Apostar" onPress={() => handleBet(10)} />
        )}
        {phase === 'jackpot' && (
          <Button title="Lanzar al Jackpot" onPress={handleJackpot} />
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  cardArea: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  fortuneArea: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  card: {
    width: 50,
    height: 70,
    margin: 5,
  },
  controls: {
    flexDirection: 'row',
  },
});

export default GameScreen;
