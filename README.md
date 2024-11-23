import List "mo:base/List";
import Option "mo:base/Option";
import Trie "mo:base/Trie";
import Nat32 "mo:base/Nat32";


actor Superheroes {
  public type SuperheroId = Nat32;

  public type Superhero = {
    name: Text;
    superpowers: List.List<Text>;
    strength: Nat32; 
    speed: Nat32;     
    intelligence: Nat32; 
  };

  private stable var next: SuperheroId = 0;
  private stable var superheroes: Trie.Trie<SuperheroId, Superhero> = Trie.empty();

  public func create(superhero: Superhero): async SuperheroId {
    let superheroId = next;
    next += 1;
    superheroes := Trie.replace(
      superheroes,
      key(superheroId),
      Nat32.equal,
      ?superhero,
    ).0;
    return superheroId;
  };

  private func key(x: SuperheroId): Trie.Key<SuperheroId> {
    return {hash = x; key = x};
  };

  public query func read(superheroId: SuperheroId): async ?Superhero {
    let result = Trie.find(superheroes, key(superheroId), Nat32.equal);
    return result;
  };

  public func update(superheroId: SuperheroId, superhero: Superhero): async Bool {
    let result = Trie.find(superheroes, key(superheroId), Nat32.equal);
    let exists = Option.isSome(result);
    if (exists) {
      superheroes := Trie.replace(
        superheroes,
        key(superheroId),
        Nat32.equal,
        ?superhero,
      ).0;
    };
    return exists;
  };

  public func battle(hero1Id: SuperheroId, hero2Id: SuperheroId): async Text {
    let hero1 = await read(hero1Id);
    let hero2 = await read(hero2Id);

    switch (hero1, hero2) {
      case (?h1, ?h2) {
        let hero1Score = h1.strength + h1.speed + h1.intelligence;
        let hero2Score = h2.strength + h2.speed + h2.intelligence;
        
        if (hero1Score > hero2Score) {
          return "Hero " # h1.name # " wins!";
        } else if (hero1Score < hero2Score) {
          return "Hero " # h2.name # " wins!";
        } else {
          return "It's a draw!";
        }
      };
      case (_, _) {
        return "One or both heroes not found.";
      };
    }
  };

  public func tournament(heroIds: List.List<SuperheroId>): async Text {
    var winners = heroIds; 
    while (List.size(winners) > 1) {
      var nextRoundWinners = List.Nil<SuperheroId>(); 

      for (i in 0..(List.size(winners) - 2) step 2) {
        let hero1 = List.nth(winners, i);
        let hero2 = List.nth(winners, i + 1);
        let result = await battle(?hero1, ?hero2);

        if (result == "Hero " # hero1.name # " wins!") {
          nextRoundWinners := List.cons(hero1, nextRoundWinners);
        } else {
          nextRoundWinners := List.cons(hero2, nextRoundWinners);
        };
      }

      winners := nextRoundWinners; 

  
    switch (List.head(winners)) {
      case (?winner) {
        let winnerHero = await read(winner);
        return "Tournament winner is: " # winnerHero.name;
      };
      case (_) {
        return "No winner found.";
      };
    }
  };
  }
};
