## PPX for job
    def crossover_job(self, pop_job):
        """
        generalisation of the precedence preservative crossover (PPX)
        """
        temp = pop_job.copy().tolist()
        new_pop = []
        for parent1 in temp:
            if random.random() < self.cross_rate:
                new_individual = []
                parent2 = pop_job[random.randint(0, self.pop_size-1)].tolist()
                string = random.choices([0, 1], k=self.chrom_size)
                for choose in string:
                    if int(choose) == 0:
                        new_individual.append(parent1[0])
                        parent2.remove(parent1[0])
                        parent1 = parent1[1:]
                    else:
                        new_individual.append(parent2[0])
                        parent1.remove(parent2[0])
                        parent2 = parent2[1:]
                new_pop.append(new_individual)
            else:
                new_pop.append(parent1)
        return np.array(new_pop)
## TPX for machine
    def crossover_ma(self, pop_machine):
        """
        two point crossover (TPX)
        """
        temp = pop_machine.copy().tolist()
        new_pop = []
        while len(temp) != 0:
            parent1, parent2 = random.sample(temp, 2)
            temp.remove(parent1)
            temp.remove(parent2)
            if random.random() < self.cross_rate:
                pos1, pos2 = sorted(random.sample(list(range(self.chrom_size)), 2))
                offspring1 = parent1[:pos1] + parent2[pos1:pos2] + parent1[pos2:]
                offspring2 = parent2[:pos1] + parent1[pos1:pos2] + parent2[pos2:]
            else:
                offspring1 = parent1
                offspring2 = parent2
            new_pop.append(offspring1)
            new_pop.append(offspring2)
        return np.array(new_pop)
## UX for machine
    def crossover_ma(self, pop_machine):
        """
        Uniform crossover (UX)
        """
        temp = pop_machine.copy().tolist()
        new_pop = []
        while len(temp) != 0:
            parent1, parent2 = random.sample(temp, 2)
            temp.remove(parent1)
            temp.remove(parent2)
            offspring1 = []
            offspring2 = []
            if random.random() < self.cross_rate:
                string = random.choices([0, 1], k=self.chrom_size)
                for i in range(self.chrom_size):
                    if int(string[i]) == 0:
                        offspring1.append(parent1[i])
                        offspring2.append(parent2[i])
                    else:
                        offspring1.append(parent2[i])
                        offspring2.append(parent1[i])
            else:
                offspring1 = parent1
                offspring2 = parent2
            new_pop.append(offspring1)
            new_pop.append(offspring2)
        return np.array(new_pop)
## POX for job
    def crossover_job(self, pop_job):
        """
        Precedence operation crossover (POX)
        """
        temp = pop_job.copy().tolist()
        new_pop = []
        job_set = list(range(self.shop.job_nb))
        random.shuffle(job_set)
        subset1, subset2 = job_set[:int(self.shop.job_nb/2)], job_set[int(self.shop.job_nb/2):]
        while len(temp) != 0:
            parent1, parent2 = random.sample(temp, 2)
            temp.remove(parent1)
            temp.remove(parent2)
            if random.random() < self.cross_rate:
                offspring1 = parent1.copy()
                order2 = [gene for gene in parent2 if gene in subset2]
                for i in range(len(offspring1)):
                    if offspring1[i] not in subset1:
                        offspring1[i] = order2[0]
                        order2 = order2[1:]
                offspring2 = parent2.copy()
                order1 = [gene for gene in parent1 if gene in subset1]
                for i in range(len(offspring2)):
                    if offspring2[i] not in subset2:
                        offspring2[i] = order1[0]
                        order1 = order1[1:]
            else:
                offspring1 = parent1
                offspring2 = parent2
            new_pop.append(offspring1)
            new_pop.append(offspring2)
        return np.array(new_pop)
## decode for FCT
    def decode(self, pop1):
        fuzzy_fitness = []
        certain_fitness = []
        for individual in pop1:
            fuzzy_completion_time, satisfaction, energy = self.shop.decode(individual)
            fuzzy_fitness.append(fuzzy_completion_time)
            certain_fitness.append(value(fuzzy_completion_time))
        return fuzzy_fitness, certain_fitness
## selection
    def selection(self, pop2, fuzzy_fitness, certain_fitness):
        """
        tournament selection + elite_strategy
        """
        pop2 = pop2.tolist()
        sorted_pop = sorted(pop2, key=lambda x: certain_fitness[pop2.index(x)], reverse=False)
        new_pop = sorted_pop[:self.elite_number]
        while len(new_pop) < self.pop_size:
            index1, index2 = random.sample(list(range(10, self.pop_size)), 2)
            if rank(fuzzy_fitness[index1], fuzzy_fitness[index2]) == fuzzy_fitness[index1]:
                new_pop.append(pop2[index2])
            else:
                new_pop.append(pop2[index1])
        return np.array(new_pop)
## elite
    @staticmethod
    def elite_strategy(pop3, fitness):
        best_fitness = [np.inf, np.inf, np.inf]
        best_individual = None
        for k, individual in enumerate(pop3):
            if rank(fitness[k], best_fitness) == best_fitness:
                best_fitness = fitness[k]
                best_individual = individual
        return best_individual, best_fitness